# 🔴 Red Hat 9 HA Cluster — Test Raporu

> **Otomatik oluşturuldu:** `ansible-playbook ha_cluster_test.yml`

---

## 📋 Genel Bilgiler

| Alan | Değer |
|------|-------|
| **Tarih / Saat** | 2026-03-25 22:41:45 |
| **Cluster Adı** | `ha_webpgsql` |
| **Node 1** | `clstr01.lab.akyuz.tech` (192.168.0.61) |
| **Node 2** | `clstr02.lab.akyuz.tech` (192.168.0.62) |
| **Quorum Device** | `clstr-qrm01.lab.akyuz.tech` (192.168.0.67) |
| **Virtual IP** | `192.168.0.63:5432` |
| **GFS2 Mount** | `/shared/webfs` |
| **PostgreSQL Mount** | `/var/lib/pgsql` |
| **Test Modu** | Yıkıcı: `True` / STONITH: `True` |

---

## 🖥️ Sistem Bilgileri

```
KERNEL=5.14.0-611.36.1.el9_7.x86_64
OS=Red Hat Enterprise Linux release 9.7 (Plow)
PACEMAKER=0.11.10
COROSYNC=
PSQL=psql (PostgreSQL) 18.3
```

**PostgreSQL:**
```
PostgreSQL 18.3 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-11), 64-bit
```

---

## ✅ Test Sonuçları Özeti

| Test | Açıklama | Sonuç | Not |
|------|----------|-------|-----|
| **T01** | Temel Cluster Sağlık Kontrolü | ✅ BAŞARILI | Her zaman çalışır |
| **T02** | VIP Failover (Node Bakımı) | ✅ BAŞARILI | Her zaman çalışır |
| **T03** | Aktif Node Ani Kapatma | ✅ BAŞARILI | `-e run_destructive=true` gerekli |
| **T04** | Corosync Ring Ağ Kesintisi | ✅ BAŞARILI | `-e run_destructive=true` gerekli |
| **T05** | Split-Brain Önleme | ✅ BAŞARILI | `-e run_destructive=true` gerekli |
| **T06** | PostgreSQL Servis Çöküşü (kill -9) | ✅ BAŞARILI | Her zaman çalışır |
| **T07** | GFS2 Eşzamanlı Yazma | ✅ BAŞARILI | Her zaman çalışır |
| **T08** | LVM Lock Doğrulama | ✅ BAŞARILI | Her zaman çalışır |
| **T09** | STONITH Fence Agent Testi | ✅ BAŞARILI (node reboot edildi) | `-e run_stonith_test=true` gerekli |
| **T10** | iSCSI Bağlantı Kesintisi | {'changed': False, 'stdout': 'Cluster name: ha_webpgsql\nCluster Summary:\n  * Stack: corosync (Pacemaker is running)\n  * Current DC: clstr01.lab.akyuz.tech (version 2.1.10-1.el9-5693eaeee) - partition with quorum\n  * Last updated: Wed Mar 25 22:36:22 2026 on clstr01.lab.akyuz.tech\n  * Last change:  Wed Mar 25 22:34:18 2026 by hacluster via hacluster on clstr01.lab.akyuz.tech\n  * 2 nodes configured\n  * 20 resource instances configured\n\nNode List:\n  * Online: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n\nFull List of Resources:\n  * fence_ipmi_clstr01.lab.akyuz.tech\t(stonith:fence_ipmilan):\t Started clstr02.lab.akyuz.tech\n  * fence_ipmi_clstr02.lab.akyuz.tech\t(stonith:fence_ipmilan):\t Started clstr01.lab.akyuz.tech\n  * Clone Set: dlm-clone [dlm]:\n    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n  * Clone Set: lvmlockd-clone [lvmlockd]:\n    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n  * vip\t(ocf:heartbeat:IPaddr2):\t Started clstr01.lab.akyuz.tech\n  * Clone Set: gfs2_lv_activate-clone [gfs2_lv_activate]:\n    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n  * Clone Set: gfs2_fs-clone [gfs2_fs]:\n    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n  * pgsql_lv_activate\t(ocf:heartbeat:LVM-activate):\t Started clstr01.lab.akyuz.tech\n  * pgsql_fs\t(ocf:heartbeat:Filesystem):\t Started clstr01.lab.akyuz.tech\n  * Clone Set: gfs2_health-clone [gfs2_health]:\n    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n  * Clone Set: php-fpm-clone [php-fpm]:\n    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n  * Clone Set: nginx-clone [nginx]:\n    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]\n  * postgresql\t(ocf:heartbeat:pgsql):\t Started clstr01.lab.akyuz.tech\n\nFailed Fencing Actions:\n  * reboot of clstr01.lab.akyuz.tech for pacemaker-controld.880745@clstr01.lab.akyuz.tech last failed at 2026-03-25 22:33:59.765237 +03:00\n  * reboot of clstr01.lab.akyuz.tech for pacemaker-controld.880745@clstr01.lab.akyuz.tech last failed at 2026-03-25 22:33:53.945113 +03:00\n\nDaemon Status:\n  corosync: active/enabled\n  pacemaker: active/enabled\n  pcsd: active/enabled', 'stderr': '', 'rc': 0, 'cmd': ['pcs', 'status'], 'start': '2026-03-25 22:36:22.219071', 'end': '2026-03-25 22:36:22.551083', 'delta': '0:00:00.332012', 'msg': '', 'stdout_lines': ['Cluster name: ha_webpgsql', 'Cluster Summary:', '  * Stack: corosync (Pacemaker is running)', '  * Current DC: clstr01.lab.akyuz.tech (version 2.1.10-1.el9-5693eaeee) - partition with quorum', '  * Last updated: Wed Mar 25 22:36:22 2026 on clstr01.lab.akyuz.tech', '  * Last change:  Wed Mar 25 22:34:18 2026 by hacluster via hacluster on clstr01.lab.akyuz.tech', '  * 2 nodes configured', '  * 20 resource instances configured', '', 'Node List:', '  * Online: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '', 'Full List of Resources:', '  * fence_ipmi_clstr01.lab.akyuz.tech\t(stonith:fence_ipmilan):\t Started clstr02.lab.akyuz.tech', '  * fence_ipmi_clstr02.lab.akyuz.tech\t(stonith:fence_ipmilan):\t Started clstr01.lab.akyuz.tech', '  * Clone Set: dlm-clone [dlm]:', '    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '  * Clone Set: lvmlockd-clone [lvmlockd]:', '    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '  * vip\t(ocf:heartbeat:IPaddr2):\t Started clstr01.lab.akyuz.tech', '  * Clone Set: gfs2_lv_activate-clone [gfs2_lv_activate]:', '    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '  * Clone Set: gfs2_fs-clone [gfs2_fs]:', '    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '  * pgsql_lv_activate\t(ocf:heartbeat:LVM-activate):\t Started clstr01.lab.akyuz.tech', '  * pgsql_fs\t(ocf:heartbeat:Filesystem):\t Started clstr01.lab.akyuz.tech', '  * Clone Set: gfs2_health-clone [gfs2_health]:', '    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '  * Clone Set: php-fpm-clone [php-fpm]:', '    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '  * Clone Set: nginx-clone [nginx]:', '    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]', '  * postgresql\t(ocf:heartbeat:pgsql):\t Started clstr01.lab.akyuz.tech', '', 'Failed Fencing Actions:', '  * reboot of clstr01.lab.akyuz.tech for pacemaker-controld.880745@clstr01.lab.akyuz.tech last failed at 2026-03-25 22:33:59.765237 +03:00', '  * reboot of clstr01.lab.akyuz.tech for pacemaker-controld.880745@clstr01.lab.akyuz.tech last failed at 2026-03-25 22:33:53.945113 +03:00', '', 'Daemon Status:', '  corosync: active/enabled', '  pacemaker: active/enabled', '  pcsd: active/enabled'], 'stderr_lines': [], 'failed': False, 'failed_when_result': False} | `-e run_destructive=true` gerekli |
| **T11** | Yük Altında Failover | ✅ BAŞARILI | Her zaman çalışır |
| **T12** | Çoklu Ardışık Failover (3x) | ✅ BAŞARILI | Her zaman çalışır |
| **T13** | Tam Cluster Yeniden Başlatma | ✅ BAŞARILI | `-e run_destructive=true` gerekli |

---

## 🏥 Son Cluster Durumu

### pcs status

```
Cluster name: ha_webpgsql
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: clstr02.lab.akyuz.tech (version 2.1.10-1.el9-5693eaeee) - partition with quorum
  * Last updated: Wed Mar 25 22:42:08 2026 on clstr01.lab.akyuz.tech
  * Last change:  Wed Mar 25 22:40:50 2026 by root via root on clstr01.lab.akyuz.tech
  * 2 nodes configured
  * 20 resource instances configured

Node List:
  * Online: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]

Full List of Resources:
  * fence_ipmi_clstr01.lab.akyuz.tech	(stonith:fence_ipmilan):	 Started clstr02.lab.akyuz.tech
  * fence_ipmi_clstr02.lab.akyuz.tech	(stonith:fence_ipmilan):	 Started clstr01.lab.akyuz.tech
  * Clone Set: dlm-clone [dlm]:
    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]
  * Clone Set: lvmlockd-clone [lvmlockd]:
    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]
  * vip	(ocf:heartbeat:IPaddr2):	 Started clstr01.lab.akyuz.tech
  * Clone Set: gfs2_lv_activate-clone [gfs2_lv_activate]:
    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]
  * Clone Set: gfs2_fs-clone [gfs2_fs]:
    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]
  * pgsql_lv_activate	(ocf:heartbeat:LVM-activate):	 Started clstr01.lab.akyuz.tech
  * pgsql_fs	(ocf:heartbeat:Filesystem):	 Started clstr01.lab.akyuz.tech
  * Clone Set: gfs2_health-clone [gfs2_health]:
    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]
  * Clone Set: php-fpm-clone [php-fpm]:
    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]
  * Clone Set: nginx-clone [nginx]:
    * Started: [ clstr01.lab.akyuz.tech clstr02.lab.akyuz.tech ]
  * postgresql	(ocf:heartbeat:pgsql):	 Started clstr01.lab.akyuz.tech

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

### Quorum Durumu

```
Quorum information
------------------
Date:             Wed Mar 25 22:42:11 2026
Quorum provider:  corosync_votequorum
Nodes:            2
Node ID:          1
Ring ID:          1.c2
Quorate:          Yes

Votequorum information
----------------------
Expected votes:   3
Highest expected: 3
Total votes:      3
Quorum:           2  
Flags:            Quorate Qdevice 

Membership information
----------------------
    Nodeid      Votes    Qdevice Name
         1          1    A,V,NMW clstr01.lab.akyuz.tech (local)
         2          1    A,V,NMW clstr02.lab.akyuz.tech
         0          1            Qdevice
```

### Corosync Ring

```
Local node ID 1, transport knet
LINK ID 0 udp
	addr	= 172.16.16.61
	status:
		nodeid:          1:	localhost
		nodeid:          2:	connected
```

---

## 🔒 Constraint'ler

```
Location Constraints:
  resource 'fence_ipmi_clstr01.lab.akyuz.tech' avoids node 'clstr01.lab.akyuz.tech' with score INFINITY (id: location-fence_ipmi_clstr01.lab.akyuz.tech-clstr01.lab.akyuz.tech--INFINITY)
  resource 'fence_ipmi_clstr02.lab.akyuz.tech' avoids node 'clstr02.lab.akyuz.tech' with score INFINITY (id: location-fence_ipmi_clstr02.lab.akyuz.tech-clstr02.lab.akyuz.tech--INFINITY)
Colocation Constraints:
  resource 'pgsql_lv_activate' with resource 'vip' (id: colo-psqllv-vip)
    score=INFINITY
  resource 'pgsql_fs' with resource 'pgsql_lv_activate' (id: colo-psqlfs-psqllv)
    score=INFINITY
  resource 'postgresql' with resource 'pgsql_fs' (id: colo-postgresql-psqlfs)
    score=INFINITY
Order Constraints:
  start resource 'dlm-clone' then start resource 'lvmlockd-clone' (id: order-dlm-lvmlockd)
  start resource 'lvmlockd-clone' then start resource 'gfs2_lv_activate-clone' (id: order-lvmlockd-gfs2lv)
  start resource 'gfs2_lv_activate-clone' then start resource 'gfs2_fs-clone' (id: order-gfs2lv-gfs2fs)
  start resource 'lvmlockd-clone' then start resource 'pgsql_lv_activate' (id: order-lvmlockd-psqllv)
  start resource 'pgsql_lv_activate' then start resource 'pgsql_fs' (id: order-psqllv-psqlfs)
  start resource 'gfs2_fs-clone' then start resource 'gfs2_health-clone' (id: order-gfs2fs-gfs2health)
  start resource 'gfs2_fs-clone' then start resource 'php-fpm-clone' (id: order-gfs2fs-phpfpm)
  start resource 'pgsql_fs' then start resource 'postgresql' (id: order-psqlfs-postgresql)
```

---

## 🗄️ Depolama Durumu

### iSCSI Session'ları

```
tcp: [1] 172.16.16.248:3260,1 iqn.2025-01.com.sirket:storage01 (non-flash)
tcp: [2] 192.168.251.248:3260,1 iqn.2025-01.com.sirket:storage01 (non-flash)
```

### Multipath Cihazlar

```
pv_ha_shared01 (360014050b56232984714db8a4bdd0ea9) dm-2 LIO-ORG,lv_ha_shared01_
size=100G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
`-+- policy='round-robin 0' prio=50 status=active
  |- 6:0:0:0 sdb 8:16 active ready running
  `- 7:0:0:0 sda 8:0  active ready running
pv_ha_shared02 (360014056d2949ca6a2b455d82d2c6b37) dm-3 LIO-ORG,lv_ha_shared02_
size=200G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
`-+- policy='round-robin 0' prio=50 status=active
  |- 7:0:0:1 sdc 8:32 active ready running
  `- 6:0:0:1 sdd 8:48 active ready running
```

### LVM Lock Durumu

```
1774467692 lvmlockd started
1774467692 No lockspaces found to adopt
1774467692 new cl 1 pi 2 fd 8
1774467692 recv vgs[916680][1] lock_sh_vg vg=db_vg  opts=0
1774467692 lockspace "lvm_db_vg" not found
1774467692 send vgs[916680][1] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close vgs[916680] cl 1 fd 8
1774467692 new cl 2 pi 2 fd 8
1774467692 recv vgs[916681][2] lock_sh_vg vg=shared_vg  opts=0
1774467692 lockspace "lvm_shared_vg" not found
1774467692 send vgs[916681][2] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close vgs[916681] cl 2 fd 8
1774467692 new cl 3 pi 2 fd 8
1774467692 recv vgs[916683][3] lock_sh_vg vg=db_vg  opts=0
1774467692 lockspace "lvm_db_vg" not found
1774467692 send vgs[916683][3] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close vgs[916683] cl 3 fd 8
1774467692 new cl 4 pi 2 fd 8
1774467692 recv vgs[916685][4] lock_sh_vg vg=shared_vg  opts=0
1774467692 lockspace "lvm_shared_vg" not found
1774467692 send vgs[916685][4] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close vgs[916685] cl 4 fd 8
1774467692 new cl 5 pi 2 fd 8
1774467692 recv vgs[916693][5] lock_sh_vg vg=db_vg  opts=0
1774467692 lockspace "lvm_db_vg" not found
1774467692 send vgs[916693][5] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close vgs[916693] cl 5 fd 8
1774467692 new cl 6 pi 2 fd 8
1774467692 recv vgs[916695][6] lock_sh_vg vg=shared_vg  opts=0
1774467692 lockspace "lvm_shared_vg" not found
1774467692 send vgs[916695][6] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close vgs[916695] cl 6 fd 8
1774467692 new cl 7 pi 2 fd 8
1774467692 recv lvs[916697][7] lock_sh_vg vg=db_vg  opts=0
1774467692 lockspace "lvm_db_vg" not found
1774467692 send lvs[916697][7] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close lvs[916697] cl 7 fd 8
1774467692 new cl 8 pi 2 fd 8
1774467692 recv lvs[916699][8] lock_sh_vg vg=shared_vg  opts=0
1774467692 lockspace "lvm_shared_vg" not found
1774467692 send lvs[916699][8] lock_sh_vg result -210 ENOLS NO_LOCKSPACES,
1774467692 close lvs[916699] cl 8 fd 8
1774467692 new cl 9 pi 2 fd 8
1774467692 recv vgchange[916751][9] lock_sh_gl   opts=0
1774467692 lockspace "lvm_global" not found for dlm gl, adding...
1774467692 add_lockspace_thread dlm lvm_global version 0
1774467692 dlm global lockspace adopt_ok
1774467692 S lvm_global lm_add_lockspace dlm wait 0 adopt_only 0 adopt_ok 1
1774467692 new cl 10 pi 3 fd 10
1774467692 recv vgchange[916767][10] lock_sh_gl   opts=0
1774467692 lockspace is starting lvm_global
1774467692 send vgchange[916767][10] lock_sh_gl result -211  
1774467692 recv vgchange[916767][10] start_vg vg=db_vg  opts=0
1774467692 add_lockspace_thread dlm lvm_db_vg version 3
1774467692 S lvm_db_vg lm_add_lockspace dlm wait 0 adopt_only 0 adopt_ok 0
1774467692 send vgchange[916767][10] start_vg result 0  
1774467692 recv vgchange[916767][10] unlock_gl   opts=0
1774467692 lockspace is starting lvm_global
1774467692 send vgchange[916767][10] unlock_gl result -211  
1774467692 recv vgchange[916767][10] start_wait   opts=0
1774467692 count_lockspace_starting client 0 count 2 done 0 fail 0
1774467692 work delayed start_wait for client 10
1774467692 count_lockspace_starting client 0 count 2 done 0 fail 0
1774467692 S lvm_db_vg lm_add_lockspace done 0
1774467693 S lvm_global lm_add_lockspace done 0
1774467693 lvm_global:GLLK action lock sh
1774467693 lvm_global:GLLK res_lock sh cl 9
1774467693 lvm_global:GLLK lock_dlm
1774467693 lvm_global:GLLK res_lock all versions zero
1774467693 lvm_global:GLLK res_lock invalidate global state
1774467693 send vgchange[916751][9] lock_sh_gl result 0  
1774467693 recv vgchange[916751][9] start_vg vg=shared_vg  opts=0
1774467693 add_lockspace_thread dlm lvm_shared_vg version 3
1774467693 S lvm_shared_vg lm_add_lockspace dlm wait 0 adopt_only 0 adopt_ok 0
1774467693 send vgchange[916751][9] start_vg result 0  
1774467693 recv vgchange[916751][9] unlock_gl   opts=0
1774467693 lvm_global:GLLK action lock un
1774467693 lvm_global:GLLK res_unlock cl 9
1774467693 lvm_global:GLLK unlock_dlm
1774467693 send vgchange[916751][9] unlock_gl result 0  
1774467693 recv vgchange[916751][9] start_wait   opts=0
1774467693 count_lockspace_starting client 0 count 1 done 2 fail 0
1774467694 work delayed start_wait for client 9
1774467694 count_lockspace_starting client 0 count 1 done 2 fail 0
1774467694 work delayed start_wait for client 10
1774467694 count_lockspace_starting client 0 count 1 done 2 fail 0
1774467694 S lvm_shared_vg lm_add_lockspace done 0
1774467696 work delayed start_wait for client 9
1774467696 count_lockspace_starting client 0 count 0 done 3 fail 0
1774467696 work delayed start_wait for client 10
1774467696 count_lockspace_starting client 0 count 0 done 3 fail 0
1774467696 send vgchange[916751][9] start_wait result 0  
1774467696 send vgchange[916767][10] start_wait result 0  
1774467696 close vgchange[916751] cl 9 fd 8
1774467696 close vgchange[916767] cl 10 fd 10
1774467696 new cl 11 pi 2 fd 8
1774467696 new cl 12 pi 3 fd 10
1774467696 recv lvchange[916837][11] lock_sh_vg vg=shared_vg  opts=0
1774467696 lvm_shared_vg:VGLK action lock sh
1774467696 lvm_shared_vg:VGLK res_lock sh cl 11
1774467696 recv lvchange[916838][12] lock_sh_vg vg=db_vg  opts=0
1774467696 lvm_db_vg:VGLK action lock sh
1774467696 lvm_db_vg:VGLK res_lock sh cl 12
1774467696 lvm_db_vg:VGLK lock_dlm
1774467696 lvm_db_vg:VGLK res_lock all versions zero
1774467696 lvm_db_vg:VGLK res_lock invalidate vg state version 0
1774467696 send lvchange[916838][12] lock_sh_vg result 0  
1774467696 lvm_shared_vg:VGLK lock_dlm
1774467696 lvm_shared_vg:VGLK res_lock all versions zero
1774467696 lvm_shared_vg:VGLK res_lock invalidate vg state version 0
1774467696 send lvchange[916837][11] lock_sh_vg result 0  
1774467696 recv lvchange[916838][12] lock_ex_lv vg=db_vg lv=pgsql_lv:qWAM1W-Rm9w-Nii4-Q1Tz-AwqE-UgW0-QCO4u2 opts=1
1774467696 lvm_db_vg:qWAM1W-Rm9w-Nii4-Q1Tz-AwqE-UgW0-QCO4u2 action lock ex
1774467696 lvm_db_vg:qWAM1W-Rm9w-Nii4-Q1Tz-AwqE-UgW0-QCO4u2 res_lock ex cl 12 (pgsql_lv)
1774467696 lvm_db_vg:qWAM1W-Rm9w-Nii4-Q1Tz-AwqE-UgW0-QCO4u2 lock_dlm
1774467696 send lvchange[916838][12] lock_ex_lv result 0  
1774467696 recv lvchange[916837][11] lock_sh_lv vg=shared_vg lv=gfs2_lv:WbvCIn-pzXy-LMBq-oviy-BzeQ-PnU3-Y7T7yp opts=1
1774467696 lvm_shared_vg:WbvCIn-pzXy-LMBq-oviy-BzeQ-PnU3-Y7T7yp action lock sh
1774467696 lvm_shared_vg:WbvCIn-pzXy-LMBq-oviy-BzeQ-PnU3-Y7T7yp res_lock sh cl 11 (gfs2_lv)
1774467696 lvm_shared_vg:WbvCIn-pzXy-LMBq-oviy-BzeQ-PnU3-Y7T7yp lock_dlm
1774467696 send lvchange[916837][11] lock_sh_lv result 0  
1774467696 recv lvchange[916837][11] unlock_vg vg=shared_vg  opts=0
1774467696 lvm_shared_vg:VGLK action lock un
1774467696 lvm_shared_vg:VGLK res_unlock cl 11
1774467696 lvm_shared_vg:VGLK unlock_dlm
1774467696 send lvchange[916837][11] unlock_vg result 0  
1774467696 close lvchange[916837] cl 11 fd 8
1774467696 recv lvchange[916838][12] unlock_vg vg=db_vg  opts=0
1774467696 lvm_db_vg:VGLK action lock un
1774467696 lvm_db_vg:VGLK res_unlock cl 12
1774467696 lvm_db_vg:VGLK unlock_dlm
1774467696 send lvchange[916838][12] unlock_vg result 0  
1774467696 close lvchange[916838] cl 12 fd 10
1774467705 new cl 13 pi 2 fd 8
1774467705 recv vgs[917955][13] lock_sh_gl   opts=0
1774467705 lvm_global:GLLK action lock sh
1774467705 lvm_global:GLLK res_lock sh cl 13
1774467705 lvm_global:GLLK lock_dlm
1774467705 send vgs[917955][13] lock_sh_gl result 0  
1774467705 recv vgs[917955][13] lock_sh_vg vg=db_vg  opts=0
1774467705 lvm_db_vg:VGLK action lock sh
1774467705 lvm_db_vg:VGLK res_lock sh cl 13
1774467705 lvm_db_vg:VGLK lock_dlm
1774467705 send vgs[917955][13] lock_sh_vg result 0  
1774467705 recv vgs[917955][13] unlock_vg vg=db_vg  opts=0
1774467705 lvm_db_vg:VGLK action lock un
1774467705 lvm_db_vg:VGLK res_unlock cl 13
1774467705 lvm_db_vg:VGLK unlock_dlm
1774467705 send vgs[917955][13] unlock_vg result 0  
1774467705 recv vgs[917955][13] lock_sh_vg vg=shared_vg  opts=0
1774467705 lvm_shared_vg:VGLK action lock sh
1774467705 lvm_shared_vg:VGLK res_lock sh cl 13
1774467705 lvm_shared_vg:VGLK lock_dlm
1774467705 send vgs[917955][13] lock_sh_vg result 0  
1774467705 recv vgs[917955][13] unlock_vg vg=shared_vg  opts=0
1774467705 lvm_shared_vg:VGLK action lock un
1774467705 lvm_shared_vg:VGLK res_unlock cl 13
1774467705 lvm_shared_vg:VGLK unlock_dlm
1774467705 send vgs[917955][13] unlock_vg result 0  
1774467705 close vgs[917955] cl 13 fd 8
1774467705 lvm_global:GLLK res_unlock cl 13 from close
1774467705 lvm_global:GLLK unlock_dlm
1774467705 new cl 14 pi 2 fd 8
1774467705 recv lvs[917956][14] lock_sh_gl   opts=0
1774467705 lvm_global:GLLK action lock sh
1774467705 lvm_global:GLLK res_lock sh cl 14
1774467705 lvm_global:GLLK lock_dlm
1774467705 send lvs[917956][14] lock_sh_gl result 0  
1774467705 recv lvs[917956][14] lock_sh_vg vg=db_vg  opts=0
1774467705 lvm_db_vg:VGLK action lock sh
1774467705 lvm_db_vg:VGLK res_lock sh cl 14
1774467705 lvm_db_vg:VGLK lock_dlm
1774467705 send lvs[917956][14] lock_sh_vg result 0  
1774467705 recv lvs[917956][14] unlock_vg vg=db_vg  opts=0
1774467705 lvm_db_vg:VGLK action lock un
1774467705 lvm_db_vg:VGLK res_unlock cl 14
1774467705 lvm_db_vg:VGLK unlock_dlm
1774467705 send lvs[917956][14] unlock_vg result 0  
1774467705 recv lvs[917956][14] lock_sh_vg vg=shared_vg  opts=0
1774467705 lvm_shared_vg:VGLK action lock sh
1774467705 lvm_shared_vg:VGLK res_lock sh cl 14
1774467705 lvm_shared_vg:VGLK lock_dlm
1774467705 send lvs[917956][14] lock_sh_vg result 0  
1774467705 recv lvs[917956][14] unlock_vg vg=shared_vg  opts=0
1774467705 lvm_shared_vg:VGLK action lock un
1774467705 lvm_shared_vg:VGLK res_unlock cl 14
1774467705 lvm_shared_vg:VGLK unlock_dlm
1774467705 send lvs[917956][14] unlock_vg result 0  
1774467705 close lvs[917956] cl 14 fd 8
1774467705 lvm_global:GLLK res_unlock cl 14 from close
1774467705 lvm_global:GLLK unlock_dlm
1774467705 new cl 15 pi 2 fd 8
1774467705 recv pvs[917957][15] lock_sh_gl   opts=0
1774467705 lvm_global:GLLK action lock sh
1774467705 lvm_global:GLLK res_lock sh cl 15
1774467705 lvm_global:GLLK lock_dlm
1774467705 send pvs[917957][15] lock_sh_gl result 0  
1774467705 recv pvs[917957][15] lock_sh_vg vg=db_vg  opts=0
1774467705 lvm_db_vg:VGLK action lock sh
1774467705 lvm_db_vg:VGLK res_lock sh cl 15
1774467705 lvm_db_vg:VGLK lock_dlm
1774467705 send pvs[917957][15] lock_sh_vg result 0  
1774467705 recv pvs[917957][15] unlock_vg vg=db_vg  opts=0
1774467705 lvm_db_vg:VGLK action lock un
1774467705 lvm_db_vg:VGLK res_unlock cl 15
1774467705 lvm_db_vg:VGLK unlock_dlm
1774467705 send pvs[917957][15] unlock_vg result 0  
1774467705 recv pvs[917957][15] lock_sh_vg vg=shared_vg  opts=0
1774467705 lvm_shared_vg:VGLK action lock sh
1774467705 lvm_shared_vg:VGLK res_lock sh cl 15
1774467705 lvm_shared_vg:VGLK lock_dlm
1774467705 send pvs[917957][15] lock_sh_vg result 0  
1774467705 recv pvs[917957][15] unlock_vg vg=shared_vg  opts=0
1774467705 lvm_shared_vg:VGLK action lock un
1774467705 lvm_shared_vg:VGLK res_unlock cl 15
1774467705 lvm_shared_vg:VGLK unlock_dlm
1774467705 send pvs[917957][15] unlock_vg result 0  
1774467705 close pvs[917957] cl 15 fd 8
1774467705 lvm_global:GLLK res_unlock cl 15 from close
1774467705 lvm_global:GLLK unlock_dlm
1774467732 new cl 16 pi 2 fd 8
1774467732 recv client[919203][16] dump_log   opts=0
```

### Disk / Mount Kullanımı

```
=== clstr01.lab.akyuz.tech ===
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/shared_vg-gfs2_lv   50G  259M   50G   1% /shared/webfs
/dev/mapper/db_vg-pgsql_lv      40G  374M   40G   1% /var/lib/pgsql
```
```
=== clstr02.lab.akyuz.tech ===
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/shared_vg-gfs2_lv   50G  259M   50G   1% /shared/webfs
/dev/mapper/rhel-root           56G   20G   37G  36% /
```

---

## 🐘 PostgreSQL

| Alan | Değer |
|------|-------|
| **VIP Bağlantısı** | ✅ Erişilebilir |
| **Host:Port** | `192.168.0.63:5432` |
| **Versiyon** | `PostgreSQL 18.3 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-11), 64-bit` |

```
   durum    |             zaman             |                                                 versiyon                                                  
------------+-------------------------------+-----------------------------------------------------------------------------------------------------------
 cluster OK | 2026-03-25 22:42:10.163737+03 | PostgreSQL 18.3 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-11), 64-bit
(1 row)
```

---

## 📊 Resource Failcount

```
No failcounts
```

---

## 📝 Test Detayları

### T01 — Temel Cluster Sağlık Kontrolü
- Corosync ring (tüm peer'ler connected)
- Quorum device bağlantısı (ffsplit algoritması)
- Cluster quorate durumu
- Tüm Pacemaker resource'ları çalışıyor
- GFS2 her iki node'da mount edilmiş
- PostgreSQL diski aktif node'da mount edilmiş
- VIP üzerinden PostgreSQL bağlantısı
- STONITH etkin
- IPMI erişilebilirliği (her node)
- Servis durumu: corosync, pacemaker, qdevice, multipathd, iscsid

### T02 — VIP Failover (Node Bakımı Simülasyonu)
- Failover öncesi PostgreSQL'e veri yazıldı
- `pcs resource move vip` ile hedef node'a taşındı
- Failover sonrası VIP üzerinden bağlantı doğrulandı
- Veri tutarlılığı: failover öncesi kayıtların korunduğu doğrulandı

### T03 — Aktif Node Ani Kapatma
- `pcs cluster stop --force` ile aktif node durduruldu
- Pasif node'da otomatik failover beklendi
- Failover sonrası bağlantı test edildi
- Failover öncesi yazılan verinin korunduğu doğrulandı
- Durdurulmuş node otomatik olarak geri başlatıldı
- **Gereksinim:** `-e run_destructive=true`

### T04 — Corosync Ring Ağ Kesintisi
- iptables ile 5405/udp corosync trafiği `45s` kesildi
- Kesinti sırasında cluster davranışı izlendi
- Ağ kurtarma sonrası corosync ring bağlantısı doğrulandı
- VIP üzerinden PostgreSQL erişilebilirliği test edildi
- **Gereksinim:** `-e run_destructive=true`

### T05 — Split-Brain Önleme
- iptables ile qdevice portu (5403/tcp) kesildi
- `no-quorum-policy=freeze` davranışı gözlemlendi
- Qdevice kurtarma ve yeniden bağlantı doğrulandı
- **Gereksinim:** `-e run_destructive=true`

### T06 — PostgreSQL Servis Çöküşü (kill -9)
- Failover öncesi veri yazıldı
- Postmaster process'e `kill -9` gönderildi
- Pacemaker `on-fail=restart` ile yeniden başlattı
- Failcount kontrol edildi (migration threshold aşılmamalı)
- Kurtarma sonrası verinin korunduğu doğrulandı

### T07 — GFS2 Eşzamanlı Yazma
- Her iki node'dan aynı anda GFS2'ye 100 satır yazıldı
- Her iki node'dan diğerinin yazdığı dosyanın okunabildiği doğrulandı
- GFS2 filesystem read-write durumu kontrol edildi

### T08 — LVM Lock Doğrulama
- lvmlockd process kontrolü
- `shared_vg` ve `db_vg` lockspace durumu doğrulandı
- lock-start idempotency testi (tekrar çalıştırılabilirlik)
- VG erişim (`vgs`) ve LV aktivasyon durumu
- `use_lvmlockd` lvm.conf konfigürasyonu

### T09 — STONITH Fence Agent Testi
- IPMI erişilebilirliği doğrulandı
- Pasif node `pcs stonith fence --reboot` ile fence edildi (async, aktif node'dan)
- Node offline duruma düştü, cluster davranışı gözlemlendi
- IPMI reboot → node otomatik açıldı, cluster'a döndü
- **Gereksinim:** `-e run_stonith_test=true` (node REBOOT eder!)

### T10 — iSCSI Bağlantı Kesintisi
- iptables ile iSCSI hedef portuna (3260/tcp) bağlantı kesildi
- Kesinti sırasında cluster davranışı izlendi
- Kurtarma sonrası GFS2 mount durumu doğrulandı
- Multipath path recovery kontrol edildi
- **Gereksinim:** `-e run_destructive=true`

### T11 — Yük Altında Failover
- 1000 INSERT işlemi arka planda başlatıldı
- Yük devam ederken VIP failover tetiklendi
- Failover tamamlandı, yazılan kayıt sayısı doğrulandı

### T12 — Çoklu Ardışık Failover (Dayanıklılık)
- `3` kez ardışık failover gerçekleştirildi
- Her döngüde veri yazılıp failover sonrası korunduğu doğrulandı
- Toplam kayıt sayısı döngü sayısıyla eşleşti

### T13 — Tam Cluster Yeniden Başlatma
- `pcs cluster stop --all` ile tüm cluster durduruldu
- 10s bekleme sonrası `pcs cluster start --all` ile başlatıldı
- GFS2 her iki node'da aktif, PostgreSQL erişilebilir durumda
- **Gereksinim:** `-e run_destructive=true`

### T14 — Web Servisi Testi (Nginx + PHP-FPM)
- VIP üzerinden `index.html` HTTP 200 doğrulandı
- VIP üzerinden `index.php` PHP-FPM erişimi doğrulandı
- `nginx_status` endpoint doğrulandı
- Her iki node'dan GFS2 web kök çapraz okuma/yazma testi
- `pcs resource cleanup nginx-clone` sonrası erişim doğrulandı
- `nginx-clone` ve `php-fpm-clone` kaynak durumu kontrol edildi

---

*Rapor `/tmp/ha_cluster_test/` dizinine kaydedildi.*
*Oluşturulma: 2026-03-25T19:41:45Z*
