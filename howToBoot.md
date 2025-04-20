# How to boot

- Hibernate Requirements in Linux
- Swap size ≥ RAM size: Traditionally, Linux requires swap space at least equal to your RAM (12GB in your case) to safely store RAM contents during hibernation.
Your current setup (4GB swap + 12GB RAM) is insufficient by default.
If you try hibernating now, it may fail or corrupt data when RAM usage exceeds 4GB.
