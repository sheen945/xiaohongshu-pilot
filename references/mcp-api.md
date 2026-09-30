# xiaohongshu-mcp 本地服务速查

> 本机部署目录：`C:\Users\Administrator\WorkBuddy\小红书相关\xiaohongshu-mcp\`
> 版本：v2.5.0（xpzouying/xiaohongshu-mcp）

## 启动

- 登录工具（首次/掉线时用，弹出扫码）：`xiaohongshu-login-windows-amd64.exe`
  - 首次运行自动下载无头浏览器（约 150MB），保持网络畅通
- MCP 服务：`xiaohongshu-mcp-windows-amd64.exe`（默认无头模式；调试加 `-headless=false`）
- 服务地址：`http://localhost:18060/mcp`（HTTP MCP 协议）
- **必须用危险模式豁免后台启动**（沙盒会回收进程），启动后 curl 验证：
  `curl http://localhost:18060/mcp` 有响应即存活
- Cookies 存于 exe 同目录，登录一次长期有效；掉线重新跑登录工具

## 13 个工具

| 工具 | 用途 | 关键参数 |
|---|---|---|
| check_login_status | 查登录状态 | 无 |
| get_login_qrcode | 取登录二维码（Base64） | 无 |
| delete_cookies | 重置登录 | 无 |
| publish_content | 发图文 | title(≤20字), content(≤1000字), images[](本地绝对路径优先，也支持 http 链接)；可选 tags[], schedule_at(ISO8601), is_original, visibility, products[] |
| publish_with_video | 发视频 | title, content, video(仅本地绝对路径)；可选同上 |
| list_feeds | 首页推荐 | 无 |
| search_feeds | 搜索 | keyword；可选 filters: sort_by(综合/最新/最多点赞/最多评论/最多收藏), note_type(图文/视频), publish_time(一天内/一周内/半年内), search_scope, location |
| get_feed_detail | 笔记详情+互动数据+评论 | feed_id, xsec_token；可选 load_all_comments, limit |
| post_comment_to_feed | 评论 | feed_id, xsec_token, content |
| reply_comment_in_feed | 回复评论 | feed_id, xsec_token, content, comment_id 或 user_id |
| like_feed | 点赞/取消 | feed_id, xsec_token, unlike |
| favorite_feed | 收藏/取消 | feed_id, xsec_token, unfavorite |
| user_profile | 用户主页数据 | user_id, xsec_token |

## 要点

- feed_id 和 xsec_token 从 search_feeds / list_feeds 结果中获取，两者缺一不可
- 定时发布：schedule_at，ISO8601 格式，范围 1 小时 ~ 14 天内
- 标题硬限 20 字、正文 1000 字，超了发布会失败
- 图片用本地绝对路径最稳；视频仅支持本地路径，建议 <1GB
- visibility：公开可见(默认)/仅自己可见/仅互关好友可见——**新笔记建议先「仅自己可见」人工检查一遍再转公开**
- AI 内容声明目前 MCP 不支持自动勾选，发布后需在 App/创作者中心手动补勾（路径：笔记 → 设置 → 内容类型声明 → 笔记含 AI 合成内容）
