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
exten => _X.,n,Hangup()```
with this
```asterisk
[trunkinbound]
exten => _X.,1,AGI(/usr/src/agi-scripts/whitelist.php,${CALLERID(num)})
exten => _X.,n,AGI(agi-DID_route.agi)
exten => _X.,n,Hangup()```

📂 Step 2: Create AGI Script Directory
cd /usr/src/
mkdir agi-scripts
📥 Step 3: Copy AGI Scripts
cd /usr/src/agi-scripts
wget https://github.com/yourusername/Inbound-Filter-for-VICIdial/raw/main/agi-scripts/whitelist.php
wget https://github.com/yourusername/Inbound-Filter-for-VICIdial/raw/main/agi-scripts/phpagi.php
wget https://github.com/yourusername/Inbound-Filter-for-VICIdial/raw/main/agi-scripts/phpagi-asmanager.php
🔑 Step 4: Set Permissions
chmod -R 755 /usr/src/agi-scripts/*.php
♻️ Step 5: Reload Dialplan
asterisk -rx "dialplan reload"
🗄️ Step 6: Setup MySQL Tables
CREATE TABLE cli_call_logs_all (
  id INT AUTO_INCREMENT PRIMARY KEY,
  caller_id VARCHAR(20),
  call_date DATE,
  call_time DATETIME,
  call_status ENUM('ALLOWED', 'BLOCKED_WHITELIST', 'BLOCKED_LIMIT') NOT NULL,
  lead_id INT DEFAULT NULL
);

CREATE TABLE cli_call_limits (
  id INT AUTO_INCREMENT PRIMARY KEY,
  caller_id VARCHAR(20),
  call_date DATE,
  call_count INT DEFAULT 0
);
⏰ Step 7: Setup Crontab Jobs
# Reset call limits daily
0 1 * * * mysql -u root asterisk -e "DELETE FROM cli_call_limits WHERE call_date < CURDATE();"

# Clean logs older than 30 days
0 1 * * * mysql -u root asterisk -e "DELETE FROM cli_call_logs_all WHERE call_date < CURDATE() - INTERVAL 30 DAY;"
🔎 Usage & Behavior
Scenario	Result
Caller NOT in whitelist	❌ Blocked
Caller in whitelist (< 5/day)	✅ Allowed
Caller in whitelist (> 5/day)	❌ Blocked
🛠️ Debugging
tail -f /tmp/whitelist.log
✅ Tested On

VICIdial 2.14 with Asterisk 16

VICIdial with Asterisk 11

ViciBox 11/12 (OpenSUSE Leap)

[trunkinbound]
exten => _X.,1,AGI(/usr/src/agi-scripts/whitelist.php,${CALLERID(num)})
exten => _X.,n,AGI(agi-DID_route.agi)
exten => _X.,n,Hangup()

