# memo


## ssh 接続が遅い問題の修正

/etc/ssh/sshd_config を修正

差分
```patch
--- sshd_config.org     2016-03-29 09:53:10.886333155 +0900
+++ sshd_config 2016-03-29 09:52:58.541605238 +0900
@@ -119,7 +119,8 @@
 #ClientAliveInterval 0
 #ClientAliveCountMax 3
 #ShowPatchLevel no
-UseDNS yes
+#UseDNS yes
+UseDNS no
 #PidFile /var/run/sshd.pid
 #MaxStartups 10:30:100
 #PermitTunnel no
```
