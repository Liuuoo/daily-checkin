# 生产级 DevOps 实践指南：从 Docker 优化到 K8s 零停机部署

在微服务架构普及的今天，运维的边界早已模糊。作为资深工程师，我见过太多团队因为一个未配置的缓存导致构建时间翻倍，或是一次不合理的资源限制导致服务在高峰期频繁 OOM。本文不谈空洞的理论，直接基于真实生产场景，分享一套经过验证的 **Docker 镜像优化 + CI/CD 自动化 + K8s 稳定部署** 实战方案。

## 一、Docker 镜像构建最佳实践：小、快、安

构建缓慢和镜像过大是新手常见的坑。核心目标是将生产镜像体积缩小 70% 以上，并消除安全隐患。

### 1. 多阶段构建（Multi-stage Builds）
不要把所有依赖都打包进最终镜像。利用构建阶段分离编译环境与运行环境。

```dockerfile
# 基于 alpine 基础镜像，体积更小
FROM node:16-alpine AS builder

WORKDIR /app

# 复制 package.json 先安装依赖，利用层缓存
COPY package*.json ./
RUN npm ci --only=production

# 复制源代码
COPY . .
RUN npm run build

# 第二阶段：运行时镜像
FROM node:16-alpine
WORKDIR /app

# 创建非 root 用户，提高安全性
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist

USER nodejs

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**关键点解析：**
*   **分层缓存策略**：先拷贝 `package*.json`，仅在依赖变化时重新执行 `npm ci`。这能极大加速 CI 流水线的构建速度。
*   **非 Root 用户**：容器内默认运行 root 用户存在安全风险。如果进程被攻破，攻击者将获得宿主机 root 权限的风险增加。
*   **Alpine 选择**：虽然 `slim` 版本对 glibc 兼容性更好，但在内存受限的微服务中，`alpine` 能节省显著资源，只需注意 musl libc 的兼容性即可。

### 2. 安全扫描集成
在 CI 流程中加入静态分析工具（如 Trivy），阻止含有高危漏洞的镜像上线。

```bash
# GitHub Actions 中的示例步骤
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'my-app:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
```

## 二、CI/CD 流水线设计：幂等性与可回滚

流水线不仅是脚本的集合，更是交付流程的标准化。我们需要确保每次发布都是**原子性**的，且失败后可快速回滚。

### 1. 推荐工作流结构
推荐使用 **GitHub Actions** 或 **Jenkins**，结合 **Helm** 进行版本管理。以下是一个简化的 Action 配置逻辑：

```yaml
name: Deploy to K8s

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      # 1. Checkout
      - uses: actions/checkout@v3

      # 2. Setup Docker
      - name: Login to Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ secrets.REGISTRY_URL }}
          username: ${{ secrets.REGISTRY_USER }}
          password: ${{ secrets.REGISTRY_PASS }}

      # 3. Build & Push
      - name: Build & Push Image
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: |
            ${{ secrets.REGISTRY_URL }}/my-service:${{ github.sha }}
            ${{ secrets.REGISTRY_URL }}/my-service:latest

      # 4. Update K8s Manifest (using Kustomize or Helm)
      - name: Deploy to K8s
        env:
          KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}
        run: |
          echo "$KUBE_CONFIG" | kubectl apply -f -
          # 等待滚动更新完成
          kubectl rollout status deployment/my-service -n production --timeout=300s
```

**实战经验：**
*   **标签规范**：使用 `${{ github.sha }}` 作为主要 Tag，保证构建的可追溯性。避免直接使用 `latest` 进行生产更新，防止回滚困难。
*   **凭据管理**：严禁硬编码密钥。必须使用 CI 平台的 Secrets 功能。
*   **并行执行**：单元测试、Lint 检查、Docker 构建应尽可能并行化，减少流水线耗时。

## 三、Kubernetes 部署策略：保障零停机与稳定性

代码发上去只是第一步，K8s 的配置决定了服务的稳定性。这里的核心是**健康检查**和**资源隔离**。

### 1. 完整的 Deployment 配置
这是生产环境必须包含的关键字段。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1      # 最多比期望副本数多 1 个实例
      maxUnavailable: 0 # 升级期间不允许服务不可用，实现零停机
      
  selector:
    matchLabels:
      app: my-service
  
  template:
    metadata:
      labels:
        app: my-service
    spec:
      containers:
      - name: app
        image: registry.example.com/my-service:v1.2.0
        ports:
        - containerPort: 3000
        
        # 关键：资源限制，防止单个 Pod 占用所有节点资源
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        # 关键：存活探针，决定重启
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        
        # 关键：就绪探针，决定是否接收流量
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

**深度解析：**
*   **Readiness vs Liveness**：这是面试必问也是实战最容易混淆的点。
    *   `livenessProbe` 失败会**重启 Pod**。适用于防止死锁（Deadlock）。
    *   `readinessProbe` 失败会将 Pod 从 Service 负载均衡列表中剔除。适用于应用启动慢或热加载期间拒绝流量。
    *   **错误案例**：很多开发者将启动命令设为 `sleep 5`，这会导致 `initialDelaySeconds` 设置过短，Service 将流量转发给尚未准备好的 Pod，引发 502 错误。
*   **资源 Requests/Limits**：
    *   `Requests` 用于调度器决定把 Pod 放在哪个节点上。
    *   `Limits` 用于防止某个服务内存泄漏拖垮整个 Node。
    *   **建议**：根据监控数据设定 `Limit`，通常设置为平均用量的 2-3 倍，预留突发处理空间。
*   **RollingUpdate 策略**：`maxUnavailable: 0` 配合 `replicas >= 2` 是实现灰度发布的基础。

### 2. 应对常见故障的实战技巧

#### 问题 A：OOMKilled (Out Of Memory Killed)
**现象**：Pod 状态变为 `CrashLoopBackOff`，日志显示 `oom-killer`。
**排查**：查看 `kubectl describe pod <pod-name> | grep -A 5 Events`。
**解决**：
1.  检查代码是否有内存泄漏。
2.  调整 `resources.limits.memory`。
3.  **重要**：开启 K8s 的 **Memory Limit Enforcement**。Java 应用需要传递 `-XX:MaxRAMPercentage=75.0` 参数，否则 JVM 不知道容器有内存限制，会尝试申请远超限制的内存导致被杀。

#### 问题 B：启动慢导致连接超时
**现象**：RollingUpdate 期间，新 Pod 还没启动完，旧 Pod 就被销毁了，造成业务中断。
**解决**：
1.  调整 `terminationGracePeriodSeconds`（默认为 30s，对于冷启动慢的服务可能需要增加到 60s+）。
2.  优化 `readinessProbe` 的逻辑，确保只有在真正准备好后才能接收请求。

#### 问题 C：数据库迁移冲突
**现象**：新版本代码发布时，旧版本的代码还在运行，但数据库 Schema 已被新版迁移破坏。
**解决**：实施 **Database Migration as a Separate Step** 策略。
1.  在 CI 流水线中，先运行一个独立的 Job 执行数据库迁移脚本。
2.  只有迁移成功后，才允许更新 Deployment。
3.  或者遵循“向前兼容”原则：新代码向下兼容旧数据库结构，待全部应用升级后再清理旧字段。

## 四、监控与可观测性闭环

没有监控的 DevOps 就是盲人摸象。除了基础的 CPU/内存，必须关注应用层面的指标。

1.  **Prometheus + Grafana**：采集 JMX/Metrics 暴露端口。
2.  **关键告警规则**：
    *   `error_rate > 1%` over 5 minutes。
    *   `latency_p99 > 2s` over 5 minutes。
    *   `container_oom_killed_total > 0`。
3.  **日志聚合**：使用 EFK (Elasticsearch, Fluentd, Kibana) 或 Loki。不要直接在容器 stdout 打印海量 Debug 日志，务必区分 Log Level。

## 五、总结与心得

构建一套稳定的 DevOps 体系不仅仅是敲几行 Yaml。它要求我们：

1.  **敬畏变更**：每一次发布都有风险，通过自动化测试和蓝绿/滚动策略降低影响面。
2.  **防御性编程**：不仅代码要健壮，基础设施配置也要考虑容错（Retries, Timeouts, Circuit Breakers）。
3.  **数据驱动**：不要拍脑袋定资源配额，一切以 Prometheus 历史监控数据为准。

**最后一条建议**：定期进行**混沌工程（Chaos Engineering）**演练。随机杀掉生产环境的 Pod，观察系统是否能自动恢复、告警是否触发。这比任何文档培训都能让你更了解系统的脆弱点。