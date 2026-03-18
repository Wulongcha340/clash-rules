# My Clash Rules

个人维护的 Clash Meta 规则集，用于多设备共享配置。

## 规则集文件

| 文件 | 用途 | behavior |
|------|------|----------|
| `ai-services.yaml` | AI 服务（OpenAI、Claude、Gemini 等） | classical |
| `academy.yaml` | 学术出版商（Nature、IEEE、ACM 等） | classical |
| `proxy.yaml` | 常见代理网站（Google、YouTube、GitHub 等） | classical |
| `direct.yaml` | 直连规则（国内常用网站、局域网、Apple/微软中国区等） | classical |

## 使用方法

### 1. 配置 rule-providers

```yaml
rule-providers:
  # 推荐：社区维护的广告规则
  reject-ad:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject-ad.yaml
    interval: 86400

  # 我的规则集
  my-ai-services:
    type: http
    behavior: classical
    url: "https://raw.githubusercontent.com/你的用户名/仓库名/main/my-rules/ai-services.yaml"
    path: ./ruleset/ai-services.yaml
    interval: 86400

  my-academy:
    type: http
    behavior: classical
    url: "https://raw.githubusercontent.com/你的用户名/仓库名/main/my-rules/academy.yaml"
    path: ./ruleset/academy.yaml
    interval: 86400

  my-proxy:
    type: http
    behavior: classical
    url: "https://raw.githubusercontent.com/你的用户名/仓库名/main/my-rules/proxy.yaml"
    path: ./ruleset/proxy.yaml
    interval: 86400

  my-direct:
    type: http
    behavior: classical
    url: "https://raw.githubusercontent.com/你的用户名/仓库名/main/my-rules/direct.yaml"
    path: ./ruleset/direct.yaml
    interval: 86400
```

### 2. 配置 rules

规则顺序很重要，按优先级从高到低排列：

```yaml
rules:
  # ============================================
  # 第一优先级：私有 IP 段直连（必须最前，使用 no-resolve）
  # ============================================
  - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve      # 本机回环
  - IP-CIDR,192.168.0.0/16,DIRECT,no-resolve   # 局域网
  - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve       # 私有网络 A 类
  - IP-CIDR,172.16.0.0/12,DIRECT,no-resolve    # 私有网络 B 类（Docker/内网）
  - IP-CIDR,17.0.0.0/8,DIRECT,no-resolve       # Apple 公司 IP 段
  - IP-CIDR,100.64.0.0/10,DIRECT,no-resolve    # 运营商级 NAT (CGNAT)
  - IP-CIDR,224.0.0.0/4,DIRECT,no-resolve      # 多播地址
  - IP-CIDR6,fe80::/10,DIRECT,no-resolve       # IPv6 链路本地

  # ============================================
  # 第二优先级：广告拦截（使用社区规则）
  # ============================================
  - RULE-SET,reject-ad,REJECT

  # ============================================
  # 第三优先级：AI 服务（优先保证代理）
  # ============================================
  - RULE-SET,my-ai-services,AI Services

  # ============================================
  # 第四优先级：直连服务（国内网站）
  # ============================================
  - RULE-SET,my-direct,DIRECT

  # ============================================
  # 第五优先级：其他代理服务
  # ============================================
  # 学术出版商
  - RULE-SET,my-academy,Academy

  # 常见代理网站（Google、YouTube、GitHub 等）
  - RULE-SET,my-proxy,Proxy

  # ============================================
  # 第六优先级：中国 IP 直连
  # ============================================
  - GEOIP,CN,DIRECT,no-resolve

  # ============================================
  # 兜底规则
  # ============================================
  - MATCH,Proxy
```

## 推荐的社区广告规则

| 项目 | URL | 说明 |
|------|-----|------|
| **Loyalsoldier** | `https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt` | 推荐，更新频繁 |
| **ACL4SSR** | `https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/BanAD.list` | 经典项目 |
| **blackmatrix7** | `https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Clash/Advertising/Advertising_Classical.yaml` | 规则最全 |

### 使用 Loyalsoldier 广告规则示例

```yaml
rule-providers:
  reject-ad:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject-ad.yaml
    interval: 86400

  reject-ad-extra:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/private.txt"
    path: ./ruleset/private.yaml
    interval: 86400
```

## 注意事项

1. **IP-CIDR 规则必须放在主配置中**，不要放入 rule-provider，以便使用 `no-resolve` 参数
2. **规则顺序**：私有 IP → 广告拦截 → AI 服务 → 直连 → 代理 → GeoIP → 兜底
3. **GEOSITE 已禁用**：本规则集使用 DOMAIN-SUFFIX 替代 GEOSITE，兼容性更好
4. **定期更新**：建议 `interval` 设为 86400（24小时）自动更新

## 代理分组建议

```yaml
proxy-groups:
  - name: Proxy
    type: select
    proxies:
      - US Auto
      - SG Auto
      - JP Auto
      - HK Auto
      - DIRECT

  - name: AI Services
    type: url-test
    url: http://www.gstatic.com/generate_204
    interval: 300
    tolerance: 100
    use:
      - YourProvider
    filter: "(?i)(?=.*(美国|US|新加坡|SG))(?=.*AI)"

  - name: Academy
    type: select
    proxies:
      - DIRECT
      - Proxy
```
