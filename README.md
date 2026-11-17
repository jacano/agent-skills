# agent-skills

Procedures an agent follows, each with the code it needs beside it. A skill is not a prompt:
it is a task that fails silently when it is done by guesswork.

| Skill | Does | Needs |
| --- | --- | --- |
| [`chrome-cdp/`](chrome-cdp/SKILL.md) | Drives the person's own Chrome — already signed in — over the DevTools protocol, and keeps the one connection Chrome allows alive. | Node 22+ |

Read a skill's `SKILL.md`, then run its scripts from the root of this repository. No
dependencies, nothing to install.

## Where this sits

| Repository | Holds |
| --- | --- |
| **agent-skills** (here) | Procedures an agent follows, with the code they need. |
| [dsh-config](https://github.com/jacano/dsh-config) | The DeepSeek Harness configuration: providers and the model menu. |
| [model-bench](https://github.com/jacano/model-bench) | The benchmark that chooses the model menu, and the page it publishes. |

## Licence

MIT. See [LICENSE](LICENSE).
