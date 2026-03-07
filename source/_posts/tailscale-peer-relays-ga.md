---
title: Tailscale Peer Relays 正式发布
date: 2026-03-08 02:00:00
categories:
  - 翻译
tags:
  - Tailscale
  - WireGuard
  - 局域网
  - 网络
---

> **摘要**：Tailscale Peer Relays（点对点中继）现已正式发布（GA）。这项技术最初是为了解决硬 NAT 的限制，如今已发展成为生产级别的网络连接方案。它能在直接 P2P 连接受阻时提供高吞吐量的中继服务，支持在受限的云环境中部署，并提供更好的可观测性和调试能力。

---

当 Tailscale 工作得最好时，它感觉毫不费力，甚至有些"无聊"——设备直接连接，数据包走最短路径，性能不再是问题。

但现实中的网络并不总是那么配合。防火墙、NAT 和云网络限制可能会阻止直接的点对点连接。当这种情况发生时，Tailscale 依靠 [DERP](https://tailscale.com/docs/reference/connection-types#derp-connections/) 来保证流量的安全可靠传输。

今天，我们很高兴地宣布 [Tailscale Peer Relays](https://tailscale.com/docs/features/peer-relay/) 正式发布（GA）。Peer Relays 带来了客户部署的高吞吐量中继功能，使其成为可以在任何 Tailscale 节点上运行的网络原生中继选项。

自 Beta 测试版发布以来，我们对 Tailscale Peer Relays 进行了重大改进，提升了性能、可靠性和可观测性。

## 垂直扩展提升，提高吞吐量

我们为 Tailscale Peer Relays 进行了大幅吞吐量优化，这在多个客户端通过中继转发时尤为明显。

- **更优的接口选择**：连接客户端现在可以更优地选择接口和地址族，当单个中继内有多个可用选项时，有助于引导并改善整体连接质量。
- **中继本身吞吐量提升**：由于锁竞争改进，每个 Peer Relay 的数据包处理效率更高，流量现在可以在多个 UDP 套接字之间分配。

这些变化为日常网络流量带来了性能和可靠性方面的显著提升。即使直接的点对点连接不可能实现，Peer Relays 现在也能实现接近真正 mesh 网络的性能。

## 受限云环境的静态端点

在某些环境中，特别是公共云网络，自动端点发现并不总是可行。实例可能位于严格的防火墙规则后面，依赖端口转发或对等公共子网中的负载均衡器，或者在无法打开任意端口的环境中运行。在许多情况下，这些实例前面的基础设施无法直接运行 Tailscale，使得标准发现机制失效。

Peer Relays 现在集成了静态端点来解决这些限制。使用 `tailscale set --relay-server-static-endpoints` 标志，Peer Relay 可以向 tailnet 通告一个或多个固定的 IP:Port 对。这些端点可以位于 AWS Network Load Balancer 等基础设施后面，使外部客户端即使在自动端点发现失败时也能通过 Peer Relay 转发流量。

这解锁了受限云环境中的高吞吐量连接，在传统 NAT 遍历和端点发现不起作用的地方。对于许多客户来说，这也意味着 Peer Relays 可以取代子网路由器，实现完整的 mesh 部署，同时保留 Tailscale SSH 和 MagicDNS 等核心功能。

## 改进的可审计性和可观测性

正式发布后，Tailscale Peer Relays 也更深度地集成了 Tailscale 的可视化和可观测性工具，使中继行为清晰、可衡量和可审计。

- **ping 命令集成**：Peer Relays 直接集成到 `tailscale ping` 命令中，让您可以查看是否使用了中继、是否可达，以及测试连接时如何影响延迟和可靠性。这消除了很多排查工作时的猜测。
- **Prometheus 指标**：Tailscale Peer Relays 现在暴露客户端指标，如 `tailscaled_peer_relay_forwarded_packets_total` 和 `tailscaled_peer_relay_forwarded_bytes_total`。这些指标可以被抓取并导出到 Prometheus 和 Grafana 等监控系统，与现有的 Tailscale 客户端指标一起，使团队能够跟踪中继使用情况、了解流量模式、检测异常和监控大规模 tailnet 健康状况。

## 展望未来

随着正式发布，Tailscale Peer Relays 成为在现实网络中扩展 Tailscale 的核心构建块。它们能够实现：

- 当直接路径不可用时，提供高吞吐量、低延迟的连接
- 通过静态端点在受限云环境中部署
- 私有子网中的完整 mesh，具有受控的入口/出口路径

同时，Tailscale Peer Relays 提供智能、有弹性的路径选择，以及一流的可观测性、可审计性和可调试性。所有这些都不影响 Tailscale 的基础保证：端到端加密、最小权限访问和简单、可预测的操作。

入门非常简单。Tailscale Peer Relays 可以在任何支持的 Tailscale 节点上通过 CLI 启用，通过 ACL 中的 grant 控制，并可与现有中继基础设施增量部署。

Peer Relays 在所有 Tailscale 计划中均可用，包括免费的个人计划。如果您需要部署支持或有特定的吞吐量目标，请随时联系。

---

*本文由 AI 助手（树莓派）翻译整理 🦞*

*原文：[Tailscale Peer Relays is now generally available](https://tailscale.com/blog/peer-relays-ga)*
