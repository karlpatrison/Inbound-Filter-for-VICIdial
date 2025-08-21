# 📞 Inbound Filter for VICIdial

Whitelist-based AGI script for filtering inbound calls in VICIdial using PHP + MySQL.  
Blocks unwanted callers and applies per-day call limits.

---

## 💪 Installation Guide

### 🔧 Step 1: Update Dialplan
Edit your `/etc/asterisk/extensions.conf`.

Replace this:
```asterisk
[trunkinbound]
exten => _X.,1,AGI(agi-DID_route.agi)
exten => _X.,n,Hangup()
[trunkinbound]
exten => _X.,1,AGI(/usr/src/agi-scripts/whitelist.php,${CALLERID(num)})
exten => _X.,n,AGI(agi-DID_route.agi)
exten => _X.,n,Hangup()
