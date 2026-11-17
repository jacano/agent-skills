# agent-skills

Procedures an agent can follow, each with the code it needs next to it. A skill here is
not a prompt: it is a task that fails in a specific, silent way when it is done by
guesswork, written down so that the next session gets it right the first time.

| Skill | Does | Needs |
| --- | --- | --- |
| [`chrome-cdp/`](chrome-cdp/SKILL.md) | Drives the person's own Chrome — already signed in — over the DevTools protocol, and keeps the one connection Chrome allows alive. | Node 22+ |

## Use one

Each directory is self-contained: read its `SKILL.md`, then run the scripts from the root
of this repository.

```bash
node chrome-cdp/cdp.mjs start
node chrome-cdp/cdp.mjs ping
```

No dependencies, nothing to install. The scripts keep their state in the system temp
directory, so they work from any clone.

## Where this sits

| Repository | Holds |
| --- | --- |
| **agent-skills** (here) | What an agent should do, and the code that does it. |
| [`dsh-config`](https://github.com/jacano/dsh-config) | The DeepSeek Harness configuration: providers and the model menu. |
| [`model-bench`](https://github.com/jacano/model-bench) | The benchmark that chooses those models, and the page it publishes. |

## Licence

MIT. See [LICENSE](LICENSE).
