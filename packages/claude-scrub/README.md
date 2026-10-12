# @magpie-community/middleware-claude-scrub

Scrub **Claude Code's** own tracks on the way out, for upstreams that turn its requests away. Google's **Antigravity** answers Claude Code chats with a disguised 429 ("Resource has been exhausted") whatever quota is left; what trips it is the Anthropic billing metadata block Claude Code puts first in `system` — `x-anthropic-billing-header: cc_version=…` — bookkeeping for Anthropic, useless to any other upstream (ablation-tested: drop the block and the same request goes through; keep it and it is turned away every time — headers, `metadata.user_id`, tools and streaming were all innocent).

So the middleware drops those blocks, and as a second layer zero-width scrambles client-identity words the way CLIProxyAPI's antigravity `sensitive-words` does: the text reads the same to the model, the filter no longer matches.

Install it from magpie's **Plugins › Discover**, or with `magpie plugin add @magpie-community/middleware-claude-scrub`. Set its options under **Plugins › Installed › Options**, or with `magpie plugin options claude-scrub '<json>'`.

## Options

```json
{ "words":   ["Claude Agent SDK", "Claude Code"],
  "billing": ["x-anthropic-billing-header"],
  "probe":   false }
```

- **`words`.** Words zero-width scrambled wherever they appear in the system prompt and messages — user text, tool results included — matched ignoring case. Replace the list with your own, or `[]` to disable the layer.
- **`billing`.** Markers of the system blocks to drop; a block whose text contains one goes. `[]` disables the layer.
- **`probe`.** `true` turns a user message of exactly `PROBE_CLAUDE_SCRUB` into a 418, proving the middleware is loaded. Off by default.

Scrambled text reads the same to the model (zero-width characters carry no meaning), so nothing behaves differently — the request diff is just invisible to the eye.

Unlike [word-guard](../word-guard), which reads only what the user writes, this one exists because Claude Code's own system prompt, not the user's text, is what needs scrubbing. The two can be installed together.

Scope: for routing your own subscription accounts through your own gateway. One gateway, one user.

中文：给会拒绝 Claude Code 的上游（如 Antigravity 的伪装 429）洗掉它的痕迹——丢掉计费元数据系统块，把身份词零宽混淆后再发出，模型读着不变。
