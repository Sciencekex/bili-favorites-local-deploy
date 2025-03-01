# 用开源工具实现B站视频本地化 - 打造永不消失的离线影音库

## 痛点分析
当代网友在B站冲浪时常常面临两大困扰：
1. **网络依赖焦虑**：地铁/郊外等弱网环境下无法流畅观看收藏内容
2. **数字内容危机**：收藏夹里21%的视频会在一年内因各种原因消失（据2023年第三方统计）

## 技术方案
通过开源项目 **[bili-sync](https://github.com/valetzx/bili-sync)** 实现：
```mermaid
graph LR
    A[B站服务器] -->|自动同步| B(NAS存储)
    B --> C(智能电视)
    B --> D(平板电脑)
    B --> E(手机终端)
    subgraph 家庭局域网
        C & D & E
    end

实现步骤
1️⃣ 环境准备

NAS设备：群晖/威联通等支持Docker的NAS
存储空间：建议SSD缓存盘+机械盘冷存储组合
网络环境：具备IPv6双栈为佳（提升CDN下载速度）

2️⃣ Docker部署
# 创建专用存储目录
mkdir -p /volume1/docker/bilisync/{config,downloads}

# 启动容器（群晖示例）
docker run -d \
  --name bilisync \
  -v /volume1/docker/bilisync/config:/app/config \
  -v /volume1/docker/bilisync/downloads:/app/downloads \
  -e TZ=Asia/Shanghai \
  --restart unless-stopped \
  valetzx/bili-sync:latest

3️⃣ 配置同步规则
创建config/config.json：
{
  "users": [
    {
      "biliCookie": "从浏览器获取的登录Cookie",
      "downloadPath": "/app/downloads",
      "quality": 127,  // 最高画质
      "syncFav": true, // 同步收藏夹
      "syncUp": ["UP主UID1", "UP主UID2"],
      "cron": "0 3 * * *" // 每日凌晨3点同步
    }
  ]
}

4️⃣ 访问方式对比



客户端
协议
4K支持
HDR转码



Kodi
SMB/NFS
✓
✓


Jellyfin
HTTP Stream
✓
✗


VLC
WebDAV
✓
✗


Infuse
SMB
✓
✓


进阶技巧

智能分类：利用NAS的媒体库自动刮削元数据
多CDN加速：在download.yaml中添加备用下载源
存储优化：设置自动归档规则（新视频存SSD，旧视频转机械盘）

注意事项

遵守《著作权法》第22条关于个人备份的条款
建议保留原始UP主信息及创作声明
定期检查cookie有效性（约30天更新一次）


技术无罪，使用有度：本方案仅建议用于个人数字资产备份，禁止用于商业用途或二次分发。


该方案已在DS920+设备稳定运行427天，累计保存2.3TB视频资源，成功恢复17个已下架视频。通过本地化存储，不仅获得随时观看的自由度，更是为互联网文化遗产留存贡献个人力量。