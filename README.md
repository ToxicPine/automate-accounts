# Prompt-Native Social Automation Skills

This repo is a small set of agent-browser skills for coding agents like Codex and Claude Code. They help automate LinkedIn, Reddit, and X from a real Chrome session using plain prompts, which may be more effective than using workflow=building tools.

The intended setup is simple: use these skills with your own prompt file and a long-running coding-agent feature such as Codex `/goal`, Claude Code `/goal`, or Claude Code `/loop`. `/goal` keeps working toward a verifiable completion condition; `/loop` reruns a prompt on an interval. The agent can browse, act, verify results, take notes, spawn subagents, and gradually learn better operating patterns as it works on your behalf.

Because this is built on `agent-browser`, any computer with Chrome can become an automation machine. For separate IPs, cookies, accounts, or browser profiles, use separate real computers or profiles rather than pretending one hosted session is many users. To coordinate multiple devices remotely, Tailscale is a good fit for exposing Chrome DevTools Protocol access privately over the internet. If you prefer hosted infrastructure, `agent-browser` can also work with cloud/SaaS browser platforms.

The goal is to be free, minimal, and approachable for people who already pay for coding agents. The skills provide the mechanics; policy choices like retries, pacing, and whether to take an action are promptable. The ability to achieve long-running execution comes from the coding harness itself, while the platform skills provide compact, script-first browser automation for LinkedIn, Reddit, and X.
