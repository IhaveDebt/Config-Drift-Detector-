
---

## 3 — Config Drift Detector (`src/config_drift_detector.py`)

**Code**
```python
#!/usr/bin/env python3
"""
Config Drift Detector
- Compare JSON/YAML configurations across environments and report diffs
Run: python3 src/config_drift_detector.py <fileA> <fileB>
"""
import json, sys
from typing import Any

def load(path):
    with open(path, "r", encoding="utf8") as f:
        txt = f.read().strip()
        if txt.startswith("{") or txt.startswith("["):
            return json.loads(txt)
        # fallback naive: treat as key=value lines
        obj = {}
        for line in txt.splitlines():
            if "=" in line:
                k,v = line.split("=",1)
                obj[k.strip()]=v.strip()
        return obj

def diff(a:Any,b:Any, prefix=""):
    if isinstance(a, dict) and isinstance(b, dict):
        keys = sorted(set(a.keys())|set(b.keys()))
        for k in keys:
            yield from diff(a.get(k), b.get(k), prefix + f"{k}.")
    else:
        if a != b:
            yield (prefix.rstrip("."), a, b)

def main():
    if len(sys.argv) < 3:
        print("Usage: config_drift_detector.py old.json new.json")
        return
    a = load(sys.argv[1]); b = load(sys.argv[2])
    for k,va,vb in diff(a,b):
        print(f"DIFF: {k} -> old={va!r} new={vb!r}")

if __name__ == "__main__":
    main()
