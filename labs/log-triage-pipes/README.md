# Log Triage with Pipes and Text Processing

## 🎯 Goal
Learn how to perform initial log triage using standard Linux CLI tools
and data pipelines.

The objective is to extract meaningful signals from raw log files
without using specialized security tools.

---

## 🧠 Concept
Instead of memorizing individual commands, this lab focuses on
building processing pipelines:

SOURCE → FILTER → EXTRACT → COUNT → SORT → LIMIT

Logs are treated as data streams, not static files.

---

## 🧪 Environment
- OS: Linux
- Tools: cat, less, grep, cut, sort, uniq, wc, head, tail
- Input: server log file (e.g. access.log / auth.log)
- Scope: local file, educational use only

---

## 🛠️ Steps

### 1. Inspect the log file
Used `cat` and `less` to understand the structure of the log
and identify relevant fields (IP, user, request).

### 2. Filter relevant entries
Applied `grep` to isolate events of interest
(e.g. authentication failures or specific services).

### 3. Extract fields
Used `cut` to extract IP addresses and usernames (when applicable).

### 4. Count occurrences
Combined `sort` and `uniq -c` to count repeated values.

### 5. Rank results
Used `sort -nr` and `head` to identify the top 10 IPs/users.

---

## 🔍 Example Pipelines

### Top 10 IP addresses
```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -nr | head

## 🧠 Observations, Conclusions & Limitations

- A small number of IPs often generate a disproportionate amount of events
- Repeated patterns are easily detectable with basic UNIX pipelines
- Correct ordering of tools (especially `sort` before `uniq`) is critical
- Effective log triage is more about thinking in data flows than individual tools

This lab demonstrates that meaningful security signals can be extracted
from raw logs without complex software, relying instead on understanding
how data moves through processing pipelines.

All actions were performed on local or test log files in a controlled
environment for educational purposes only.

