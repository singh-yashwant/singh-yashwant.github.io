---
title: "Mermaid Sample"
date: 2023-04-09T22:49:39+05:30
draft: true
mermaid: true
---

{{< mermaid >}}
flowchart LR
    y("👫 You") --> h{"🤝 Found this helpful?"}
    h --> |Yes| r[/"⭐ Check out my featured posts!"/]
    h --> |No| su[/"📝 Suggest changes by clicking near the title"/]
    click r "/categories/featured" _blank
{{< /mermaid >}}