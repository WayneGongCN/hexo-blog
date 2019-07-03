---
title: Firefox 隐私设置
date: 2018-05-05
tags: privacy
categories: notes
---

## 浏览器安全隐私安全测试
[Panopticlick](https://panopticlick.eff.org/)

## FireFox 的隐私设置调整
进入 `about:config` 进行如下修改
1. privacy.firstparty.isolate = true
2. privacy.resistFingerprinting = true
3. privacy.trackingprotection.enabled = true
4. browser.cache.offline.enable = false
5. browser.safebrowsing.malware = false
6. browser.safebrowsing.phishing.enabled = false
7. browser.send_pings = false
8. browser.sessionstore.max_tabs_undo = 0
9. browser.urlbar.speculativeConnect.enabled = false
10. dom.battery.enabled = false
11. dom.event.clipboardevents.enabled = false
12. geo.enabled = false
13. media.navigator.enabled = false
14. network.cookie.cookieBehavior = 1
15. etwork.cookie.lifetimePolicy = 2
16. network.http.referer.trimmingPolicy = 2
17. network.http.referer.XOriginPolicy = 2
18. network.http.referer.XOriginTrimmingPolicy = 2
19. webgl.disabled = true

## FireFox 取消 WebRTC
进入 `about:config` 进行如下修改
1. media.peerconnection.enabled = false
2. media.peerconnection.turn.disable = true
3. media.peerconnection.use_document_iceservers = false
4. media.peerconnection.video.enabled = false
5. media.peerconnection.identity.timeout = 1

## FireFox 隐私保护插件
1. Privacy Badger 阻止不遵守DNT协议广告商的跟踪行为。
2. uBlock Origin 内容过滤浏览器扩展
3. Self-Destructing Cookies 自动删除 Cookies
4. HTTPS Everywhere 对支持 HTTPS 的网站启用加密连接
5. Decentraleyes 使用本地CDN库文件并拦截第三方库文件来提高网页加载速度并保护隐私。
6. uMatrix 更加高级（复杂）的内容过滤工具。
7. NoScript Security Suite 选择让 Javascript、Java 与 Flash 只能在所信任的网站上执行。