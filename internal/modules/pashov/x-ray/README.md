# X-Ray

Know your protocol before auditors do.

Built for:

- **Protocol teams** preparing for an audit G현 fix the obvious so auditors can focus on what matters
- **Security researchers** starting a new engagement G현 get the full picture in minutes

Not a vulnerability scanner G현 it's the briefing you read before opening the first file.

## What You Get

One command produces:

| Output | What's Inside |
|--------|--------------|
| `x-ray.md` | Protocol overview, threat model, test gaps, git history, readiness verdict |
| `entry-points.md` | Every state-changing function classified by access level with call chains |
| `invariants.md` | Full invariant map G현 enforced guards, single-contract invariants, cross-contract trust assumptions, and higher-order economic properties |
| `architecture.svg` | Visual architecture diagram G현 contracts, actors, trust boundaries |

## Demo

_Part of an X-Ray report generation shown below_

![Running x-ray in terminal](../static/x_ray.gif)

## Usage

```
Install latest https://github.com/pashov/skills/ and run x-ray on the codebase
```

## Tips

- **Start with the verdict.** The report ends with a tier (FORTIFIED G樣 EXPOSED) and 3-5 action items. If you only read one section, read that.
- **Use entry-points.md as your map.** Start with permissionless functions G현 those are the highest-risk surface.
- **Check the action items.** The final section highlights concrete next steps G현 whether you're preparing for an audit or starting one.
