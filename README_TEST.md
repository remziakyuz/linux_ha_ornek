# Red Hat 9 HA Cluster — Test Playbook

**`ha_cluster_test.yml`** · RHEL 9 · Pacemaker 2.1.x · Corosync 3.x · PostgreSQL 18 · Nginx · PHP-FPM

> Bu döküman `ha_cluster_test.yml` playbook'unu daha önce hiç kullanmamış birinin testleri başarıyla çalıştırabilmesi için hazırlanmıştır. Her testin ne yaptığı, nasıl çalıştırıldığı, hangi koşulları doğruladığı ve sonuçların nasıl yorumlandığı eksiksiz olarak açıklanmıştır.
>
> Test playbook'u **yalnızca** `ha_cluster_setup.yml` ile başarıyla tamamlanmış bir cluster kurulumundan sonra çalıştırılmalıdır.

---

## İçindekiler

1. [Genel Bakış](#1-genel-bakış)
2. [Ön Koşullar](#2-ön-koşullar)
3. [Dizin Yapısı](#3-dizin-yapısı)
4. [Kullanım](#4-kullanım)
5. [PREP — Test Öncesi Hazırlık](#5-prep--test-öncesi-hazırlık)
6. [Test Detayları](#6-test-detayları)
7. [RAPOR — Son Doğrulama ve Ortam Restore](#7-rapor--son-doğrulama-ve-ortam-restore)
8. [Markdown Rapor Dosyası](#8-markdown-rapor-dosyası)
9. [Güvenlik ve Temizlik Mekanizması](#9-güvenlik-ve-temizlik-mekanizması)
10. [Test Hatalarını Giderme](#10-test-hatalarını-giderme)

---

## 1. Genel Bakış

Test playbook'u cluster'ın tüm başarısızlık senaryolarında doğru davranıp davranmadığını otomatik olarak doğrular. Testler iki gruba ayrılır:

### Güvenli Testler (her zaman çalışır)

| Test | Açıklama |
|---|---|
| T01 | Temel cluster sağlık kontrolü (**nginx, php-fpm, http, php-pgsql kontrolleri dahil**) |
| T02 | VIP failover (node bakımı simülasyonu) |
| T06 | PostgreSQL servis çöküşü (kill -9) |
| T07 | GFS2 eşzamanlı yazma (her iki node) |
| T08 | LVM lock doğrulaması |
| T11 | Yük altında failover |
| T12 | Çoklu ardışık failover (dayanıklılık) |
| **T14** | **Web servisi testi (Nginx + PHP-FPM) — kapsamlı** |

### Yıkıcı Testler (`-e run_destructive=true` gerekli)

| Test | Açıklama | Risk |
|---|---|---|
| T03 | Aktif node ani kapatma | Node durdurulup yeniden başlatılır |
| T04 | Corosync ring ağ kesintisi | iptables kuralları uygulanır |
| T05 | Split-brain önleme (qdevice engelleme) | iptables kuralları uygulanır |
| T10 | iSCSI bağlantı kesintisi (`configure_iscsi=true` ise) | iptables kuralları uygulanır |
| T13 | Tam cluster yeniden başlatma | Tüm node'lar durdurulup başlatılır |

### STONITH Testi (`-e run_stonith_test=true` gerekli)

| Test | Açıklama | Risk |
|---|---|---|
| T09 | STONITH fence agent testi | **Node IPMI üzerinden REBOOT edilir!** |

---

## 2. Ön Koşullar

- `ha_cluster_setup.yml` ile cluster kurulumu başarıyla tamamlanmış olmalı
- Her iki node online: `pcs status` çıktısında OFFLINE veya FAILED resource yok
- PostgreSQL VIP üzerinden erişilebilir: `psql -h 192.168.0.63 -U postgres -c "SELECT 1;"`
- `pgsql_password` değişkeni `inventory/hosts.yml`'de doğru tanımlı
- `tasks/failover_cycle.yml` dosyası proje dizininde mevcut (T12 için)
- **Web servisi için:** `php-pgsql` paketi her iki node'da kurulu olmalı (`dnf install php-pgsql -y`)
- **Web servisi için:** nginx resource `httpd=/usr/sbin/nginx` parametresiyle tanımlı olmalı

### Test Öncesi Cluster Sağlık Kontrolü

```bash
# Cluster temiz mi?
pcs status
# Beklenen: tüm resource'lar Started, FAILED veya OFFLINE yok

# Stale failure counter'ları temizle
pcs resource cleanup

# PostgreSQL erişilebilir mi?
psql -h 192.168.0.63 -U postgres -c "SELECT version();"

# Web servisi hazır mı?
curl -s -o /dev/null -w "%{http_code}" http://192.168.0.63/index.html  # 200 beklenir
curl -s -o /dev/null -w "%{http_code}" http://192.168.0.63/index.php   # 200 beklenir

# php-pgsql kurulu mu?
rpm -qa | grep php-pgsql  # Her iki node'da

# nginx httpd parametresi set mi?
pcs resource config nginx | grep httpd  # httpd=/usr/sbin/nginx görmeli# pgsql_password doğru mu?
PGPASSWORD="hosts.yml_deki_parola" psql -h 192.168.0.63 -U postgres -c "SELECT 1;"
```

---

## 3. Dizin Yapısı

```
proje-dizini/
├── ha_cluster_test.yml           # Ana test playbook'u
└── tasks/
    └── failover_cycle.yml        # T12 failover döngüsü task dosyası
```

`tasks/failover_cycle.yml` dosyası T12 tarafından `include_tasks` ile çağrılır. `ha_cluster_test.yml` ile aynı dizindeki `tasks/` alt klasöründe bulunmalıdır.

---

## 4. Kullanım

### Tüm Testleri Çalıştır (Güvenli Mod)

```bash
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml
```

Yıkıcı testler (T03, T04, T05, T10, T13) ve STONITH testi (T09) bu modda **atlanır**.

### Yıkıcı Testler Dahil

```bash
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml \
  -e run_destructive=true
```

### STONITH Testi Dahil (Node Reboot Olur!)

```bash
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml \
  -e run_destructive=true \
  -e run_stonith_test=true
```

### Tek Test Çalıştır

```bash
# Sadece T01
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml -t test_T01

# Sadece T06
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml -t test_T06

# Sadece T09 (STONITH — node reboot!)
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml \
  -t test_T09 -e run_stonith_test=true
```

### Belirli Test Seti Çalıştır

```bash
# T01 ve T02
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml \
  -t "test_T01,test_T02"

# Sadece güvenli testler
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml \
  -t "test_T01,test_T02,test_T06,test_T07,test_T08,test_T11,test_T12"

# Yıkıcı testler hariç tümü (T09 dahil değil)
ansible-playbook -i inventory/hosts.yml ha_cluster_test.yml \
  -t "test_T03,test_T04,test_T05,test_T10,test_T13" \
  -e run_destructive=true
```

### Değişken Referansı

| Değişken | Varsayılan | Açıklama |
|---|---|---|
| `run_destructive` | `false` | Yıkıcı testleri etkinleştir (T03, T04, T05, T10, T13) |
| `run_stonith_test` | `false` | STONITH testini etkinleştir (T09) — node reboot olur! |
| `failover_wait` | `60` | Failover tamamlanması için bekleme süresi (saniye) |
| `settle_wait` | `30` | Resource stabilizasyonu için bekleme süresi (saniye) |
| `pgsql_password` | (hosts.yml'den) | PostgreSQL postgres kullanıcısı parolası — ayrı tanım gerekmez |

> **Not:** `pgsql_password` değişkeni `inventory/hosts.yml`'deki değerden otomatik alınır. Tüm play'lerde inventory group_vars üzerinden erişilir; ayrıca tanımlanmasına gerek yoktur.

---

## 5. PREP — Test Öncesi Hazırlık

Her test çalışmadan önce PREP play'i şu temizlik ve doğrulama adımlarını otomatik olarak gerçekleştirir:

### Önceki Test Artıklarını Temizleme

1. **iptables temizliği:** Tüm cluster node'larından önceki test çalışmalarına ait iptables kuralları kaldırılır (corosync 5405, qdevice 5403, iSCSI 3260 port kuralları)
2. **PostgreSQL tablo temizliği:** Kalan test tabloları drop edilir: `ha_test_t02`, `ha_test_t03`, `ha_test_t06`, `ha_load_t11`, `ha_test_t12`
3. **Location constraint temizliği:** VIP üzerindeki tüm geçici kısıtlamalar kaldırılır (`pcs resource clear vip`)
4. **Resource cleanup:** `pcs resource cleanup` ile stale failure counter'lar sıfırlanır
5. **Stabilizasyon bekleme:** 15 saniye beklenir

### Cluster Sağlık Doğrulaması

6. **`pcs status` kontrolü:** OFFLINE veya FAILED resource varsa test durur
7. **PostgreSQL bağlantı testi:** `pgsql_password` ile VIP üzerinden bağlantı test edilir — parola yanlışsa T01'de değil, burada net hata verir
8. **Aktif/pasif node belirleme:** VIP'in hangi node'da olduğu tespit edilir ve ekrana yazdırılır

### Test Ortamı Özeti

PREP sonunda şu bilgiler ekrana basılır:

```
╔══════════════════════════════════════════════════════════════╗
║          TEST ORTAMI HAZIR                                   ║
╠══════════════════════════════════════════════════════════════╣
║  Cluster     : ha_webpgsql
║  Aktif Node  : clstr01.lab.akyuz.tech
║  Pasif Node  : clstr02.lab.akyuz.tech
║  VIP         : 192.168.0.63
║  PostgreSQL  : 192.168.0.63:5432
║  GFS2 Mount  : /shared/webfs
║  pgsql Mount : /var/lib/pgsql
║  Yıkıcı test : DEVRE DIŞI (güvenli mod)
║  STONITH test: DEVRE DIŞI
╚══════════════════════════════════════════════════════════════╝
```

---

## 6. Test Detayları

### T01 — Temel Cluster Sağlık Kontrolü

**Tag:** `test_T01` | **Tür:** Güvenli | **Her zaman çalışır:** Evet

Bu test cluster'ın bütüncül sağlığını doğrular. Diğer testler çalıştırılmadan önce bu testin geçmesi beklenir.

**Yapılan kontroller:**

| Kontrol | Komut | Beklenen Sonuç |
|---|---|---|
| Corosync ring | `corosync-cfgtool -s` | Tüm peer'ler connected, disconnected/lost yok |
| Quorum device | `pcs quorum device status` | State: connected |
| Cluster quorate | `pcs quorum status` | Quorate: Yes |
| Tüm resource'lar | `pcs resource status` | FAILED veya Stopped yok |
| GFS2 her iki node'da | `pcs resource status gfs2_fs-clone` | Her iki node adı çıktıda var |
| GFS2 mount (her node) | `mountpoint -q /shared/webfs` | rc=0 her iki node'da |
| PostgreSQL mount | `mountpoint -q /var/lib/pgsql` | rc=0 aktif node'da |
| PostgreSQL bağlantısı | `psql -h VIP -c "SELECT version()"` | Başarılı |
| STONITH etkin | `pcs property config stonith-enabled` | stonith-enabled=true |
| IPMI erişilebilirliği | `ipmitool chassis power status` | Her node için rc=0 |
| Yapılandırma doğrulaması | `crm_verify -L -V` | rc=0 |
| Servis durumu | `systemctl is-active` | corosync, pacemaker, corosync-qdevice, multipathd, iscsid hepsi active |

**Hata durumunda:** Temel cluster sorunu çözülmeden diğer testlere geçilmemelidir.

---

### T02 — VIP Failover (Node Bakımı Simülasyonu)

**Tag:** `test_T02` | **Tür:** Güvenli | **Her zaman çalışır:** Evet

**Ne yapar:**

1. Hangi node'un VIP'i tuttuğunu kaydeder
2. PostgreSQL'e test kaydı yazar (`ha_test_t02` tablosu)
3. VIP'i diğer node'a taşır: `pcs resource move vip <hedef>`
4. VIP'in hedef node'a geçmesini bekler (en fazla 60s, 5s aralıklarla)
5. PostgreSQL'in yeni node'da başlamasını bekler
6. Failover sonrası ikinci kayıt ekler
7. Failover öncesi kaydın korunduğunu doğrular (`count >= 1`)
8. Location constraint kaldırır (`pcs resource clear vip`)
9. Test tablosunu drop eder

**Güvenlik:** `block/rescue/always` yapısı ile sarılmıştır. Test başarısız olsa bile location constraint kaldırılır ve test tablosu temizlenir.

**Kanıtladığı şeyler:**
- VIP failover yapılandırılmış timeout içinde tamamlanır
- PostgreSQL, failover boyunca VIP üzerinden erişilebilir kalır
- Failover öncesi yazılan veri, failover sonrasında korunur

---

### T03 — Aktif Node Ani Kapatma

**Tag:** `test_T03` | **Tür:** Yıkıcı | **Gereksinim:** `-e run_destructive=true`

**Ne yapar:**

1. Aktif node (VIP sahibi) ve pasif node'u belirler
2. PostgreSQL'e test kaydı yazar (`ha_test_t03` tablosu)
3. Aktif node'u durdurur: `pcs cluster stop <aktif_node> --force` (asenkron, 10s timeout)
4. Pasif node üzerinde PostgreSQL'in başlamasını bekler (en fazla 120s)
5. VIP'in pasif node'a geçtiğini doğrular
6. Pasif node üzerinden VIP'e bağlanarak veri tutarlılığını test eder
7. Failover öncesi kaydın korunduğunu doğrular (`count >= 1`)

**always bloğu (her durumda çalışır):**
- Durdurulan node yeniden başlatılır: `pcs cluster start <aktif_node>`
- Node'un cluster'a geri gelmesi beklenir (24 × 5s)
- `pcs resource cleanup`
- Test tablosu drop edilir

**Kanıtladığı şeyler:**
- Cluster node arızasını doğru tespit eder ve otomatik failover yapar
- Pasif node PostgreSQL'i devralır
- Ani node kapatma sonrası veri korunur
- Durdurulmuş node cluster'a başarıyla geri döner

---

### T04 — Corosync Ring Ağ Kesintisi

**Tag:** `test_T04` | **Tür:** Yıkıcı | **Gereksinim:** `-e run_destructive=true`

**Ne yapar:**

1. Aktif ve pasif node'u belirler
2. Pasif node üzerinde corosync trafiğini keser: iptables ile UDP/TCP 5405 portunu 45s boyunca engeller (asenkron)
3. 15 saniye bekler, ardından aktif node'dan tek seferlik cluster durum snapshot alır
4. Ağın geri gelmesini bekler (45 + 10s)
5. Cluster stabilizasyonunu bekler (en fazla 120s) — aşağıdaki koşulların tümü sağlanana kadar:
   - `OFFLINE` yok
   - `Failed Resource Actions` yok
   - Her iki node adı çıktıda görünüyor
   - `partition with quorum` onaylanmış
6. `pcs resource cleanup` çalıştırır
7. VIP üzerinden PostgreSQL erişilebilirliğini test eder
8. Corosync ring'in tamamen kurtarıldığını doğrular (`corosync-cfgtool -s`)

**Neden 45 saniye?** Bu süre STONITH delay değerinden (clstr02 için 30s) büyük olmalıdır. Quorum/fencing mantığını tetiklemek için yeterli süre sağlar.

> **Stabilizasyon koşulu neden kapsamlı?** Ağ kesintisi sonrası `pcs status` çıktısında clone resource'lar için `Stopped: [ clstrXX ]` satırı geçici olarak görünebilir. Yalnızca `OFFLINE` ve `Failed Resource Actions` kontrolü bu durumda yeterli değildir; `partition with quorum` ve her iki node adının çıktıda bulunması, cluster'ın gerçekten sağlıklı olduğunu doğrular.

**always bloğu:** iptables kuralları her durumda pasif node'dan kaldırılır.

**Kanıtladığı şeyler:**
- Cluster geçici corosync ring kesintisini yönetir
- Ağ kurtarma sonrası corosync ring yeniden bağlanır
- VIP ve PostgreSQL kesinti boyunca erişilebilir kalır

---

### T05 — Split-Brain Önleme (Quorum Device Engelleme)

**Tag:** `test_T05` | **Tür:** Yıkıcı | **Gereksinim:** `-e run_destructive=true`

**Ne yapar:**

1. Her iki cluster node'unda qdevice portunu (TCP 5403) iptables ile 30s boyunca engeller (asenkron)
2. Kesinti sırasında quorum durumunu izler: `corosync-quorumtool -s`
3. Beklenen davranış analizi ekrana basılır: `no-quorum-policy=freeze` ile resource'lar dondurulmalı
4. Kurtarma beklenir (30 + 15s)
5. Qdevice'ın yeniden bağlandığını doğrular (`pcs quorum device status`)

**always bloğu:** iptables kuralları her iki node'dan kaldırılır.

**Kanıtladığı şeyler:**
- Qdevice bağlantısı kesildiğinde `no-quorum-policy=freeze` devreye girer (resource'lar durdurulmaz, dondurulur)
- Qdevice bağlantısı yeniden kurulduktan sonra cluster fully quorate duruma döner
- ffsplit algoritması qdevice kaybını split-brain oluşturmadan yönetir

---

### T06 — PostgreSQL Servis Çöküşü (kill -9)

**Tag:** `test_T06` | **Tür:** Güvenli | **Her zaman çalışır:** Evet

**Ne yapar:**

1. PostgreSQL'in hangi node'da çalıştığını tespit eder
2. PostgreSQL'e test kaydı yazar (`ha_test_t06` tablosu)
3. Postmaster PID'ini tespit eder: `pgrep -o -f 'postgres.*postmaster'`
4. Postmaster'a `kill -9` gönderir
5. 5 saniye bekler (Pacemaker'ın arızayı fark etmesi için)
6. Pacemaker'ın PostgreSQL'i yeniden başlatmasını bekler (en fazla 60s, 5s aralıklarla)
7. Failcount kontrol eder — migration threshold (3) aşılmamış olmalı
8. Yeniden başlatma sonrası test kaydı ekler
9. Kill öncesi kaydın korunduğunu doğrular (`count >= 1`)

**always bloğu:** `pcs resource cleanup postgresql`, test tablosu drop edilir.

**Kanıtladığı şeyler:**
- Pacemaker'ın `on-fail=restart` monitor aksiyonu çöken PostgreSQL'i doğru yeniden başlatır
- Tek çöküşte migration threshold (3) aşılmaz
- Çöküş öncesi yazılan veri yeniden başlatma sonrasında korunur

---

### T07 — GFS2 Eşzamanlı Yazma (Her İki Node)

**Tag:** `test_T07` | **Tür:** Güvenli | **Her zaman çalışır:** Evet | **Hedef:** `cluster_nodes` (her ikisi)

**Ne yapar:**

1. Her iki node'da GFS2 mount durumunu doğrular
2. `/shared/webfs` disk alanını kontrol eder
3. Her iki node'dan eşzamanlı olarak `/shared/webfs`'e 100 satır yazar (asenkron, 60s timeout):
   - Dosya adı: `ha_test_t07_<hostname>.txt`
4. Her iki yazma işleminin tamamlanmasını bekler
5. Her iki node'dan diğer node'un yazdığı dosyanın görünebildiğini doğrular (çapraz okuma)
6. Her dosyada 100+ satır olduğunu doğrular
7. GFS2'nin `gfs2` tipiyle read-write mount edildiğini doğrular

**always bloğu:** GFS2'deki tüm test dosyaları kaldırılır (`rm -f ha_test_t07_*.txt`), run_once=true.

**Kanıtladığı şeyler:**
- GFS2 her iki node'dan eşzamanlı yazmaları bozulmadan yönetir
- Bir node'un yazdığı dosya diğer node'dan hemen görünür
- DLM üzerinden GFS2 dağıtık kilitleme doğru çalışır

---

### T08 — LVM Lock Doğrulaması

**Tag:** `test_T08` | **Tür:** Güvenli | **Her zaman çalışır:** Evet

**Ne yapar:**

1. `lvmlockd` process'inin **her iki node'da** çalışıp çalışmadığını doğrular (`pgrep -x lvmlockd`)
2. `pgsql_lv_activate` resource'unun hangi node'da çalıştığını tespit eder — bu `db_vg`'nin aktif node'udur
3. **shared_vg lockspace:** Her iki node'dan `lvmlockctl --dump` çalıştırır ve `shared_vg` lockspace'inin her node'da görünebildiğini doğrular
4. **db_vg lockspace:** Yalnızca `pgsql_lv_activate`'in çalıştığı node'dan (`t08_db_active_node`) `lvmlockctl --dump` çalıştırır ve `db_vg` lockspace'ini doğrular
5. `pcs resource failcount show lvmlockd` ile failcount raporunu gösterir
6. `lvm.conf`'taki `use_lvmlockd` değerini her iki node'dan okur ve gösterir
7. **shared_vg** için her iki node'dan `vgchange --lock-start` idempotency testi yapar
8. **db_vg** için yalnızca aktif node'dan `vgchange --lock-start` idempotency testi yapar
9. `vgs` ile VG erişim testi yapar (`shared_vg` her iki node'dan, `db_vg` aktif node'dan)
10. `lvs` ile LV aktivasyon durumunu raporlar

> **Neden db_vg farklı node'dan test edilir?**
> `db_vg`, `activation_mode=exclusive` ile tanımlıdır — LVM lockspace yalnızca `pgsql_lv_activate` resource'unun çalıştığı node'da açılır. Test başlangıcında `pcs resource status pgsql_lv_activate` ile aktif node dinamik olarak tespit edilir ve tüm `db_vg` kontrolleri bu node'a `delegate_to` ile yönlendirilir. clstr01'den yanlış node üzerinde `lvmlockctl --dump` çalıştırmak `db_vg` lockspace'ini göstermez.

**Kanıtladığı şeyler:**
- `lvmlockd` her iki node'da çalışıyor ve dağıtık lock'ları doğru yönetiyor
- `shared_vg` (clone, aktif/aktif) her iki node'da aktif lockspace'e sahip
- `db_vg` (exclusive, aktif/pasif) yalnızca doğru node'da lockspace açık
- lock-start idempotent (birden fazla çalıştırılabilir, hata vermez)
- `lvm.conf`'ta `use_lvmlockd = 1` aktif

---

### T09 — STONITH Fence Agent Testi

**Tag:** `test_T09` | **Tür:** STONITH | **Gereksinim:** `-e run_stonith_test=true`

> **UYARI:** Bu test pasif cluster node'unu IPMI üzerinden fiziksel olarak REBOOT EDER. Ortamınızda buna izin verildiğinden emin olun.

**Ne yapar:**

1. Pasif node'u (VIP'i tutmayan) ve aktif node'u belirler
2. **Aktif node'dan** pasif node'un IPMI'sini test eder: `ipmitool chassis power status`
3. **Aktif node'dan, asenkron olarak** pasif node'u fence eder: `pcs stonith fence <pasif_node> --reboot`
4. Fence işleminin tamamlandığını `async_status` ile doğrular
5. 15 saniye stabilizasyon bekler; o anki cluster durumunu snapshot alır
6. SSH portunu bekler: aktif node üzerinden `wait_for port:22` (15s delay, 180s timeout)
7. Cluster servisini başlatır: `pcs cluster start` (boot'ta otomatik başlamamışsa)
8. Node'un cluster'a döndüğünü ve quorum'un tam olduğunu doğrular (24 × 10s)

> **Neden `--reboot` (eski `--off` değil)?**
> `fence_ipmilan` kaynağı `meta pcmk_reboot_action=reboot` ile tanımlanmıştır. `--off` kullanmak fence agent ile çelişir ve node kapatıldıktan sonra otomatik açılmaz — elle `ipmitool power on` gerektirir. `--reboot` ise node'u kapatıp doğrudan açar.

> **Neden `delegate_to: aktif_node` ve `async`?**
> `pcs stonith fence` senkron çalışır ve tamamlanana kadar bloklar. Komutu fence edilecek node üzerinde çalıştırmak, node kapandığında SSH'ın düşmesine ve Ansible'ın sonsuza askıda kalmasına neden olur. `async: 120 / poll: 0` ile komut arka plana alınır; `async_status` ile sonuç hayatta kalan node'dan takip edilir.

> **Neden OFFLINE penceresi beklenmez?**
> `--reboot` ile hızlı ortamlarda (VM, IPMI simülatör) node çok kısa sürede yeniden açılır ve `pcs status` çıktısında hiç `OFFLINE` görünmeden geri dönebilir. "OFFLINE bekle" adımı bu tür ortamlarda timeout ile başarısız olur. Bunun yerine fence komutunun başarıyla tamamlandığı `async_status` ile zaten doğrulanmaktadır.

**rescue bloğu:** Hata durumunda `ipmitool chassis power on` ile node'u açmayı dener. Bu, `--reboot` başarısız olup node kapalı kaldıysa devreye girer.

**always bloğu:** `pcs resource cleanup` aktif node'dan çalışır.

**Kanıtladığı şeyler:**
- STONITH agent (`fence_ipmilan`) node'u başarıyla fence edebilir
- IPMI kimlik bilgileri ve bağlantısı doğrudur
- Fenced node kurtarılıp cluster'a yeniden katılabilir
- Tek node fence edilirken cluster (aktif node + quorum device) çalışmaya devam eder

---

### T10 — iSCSI Bağlantı Kesintisi

**Tag:** `test_T10` | **Tür:** Yıkıcı | **Gereksinim:** `-e run_destructive=true`

**Ne yapar:**

1. Aktif iSCSI session'larını ve multipath durumunu kaydeder (kesinti öncesi)
2. Her iki cluster node'undan iSCSI hedefini 30s boyunca iptables ile engeller (asenkron):
   - `iptables -I OUTPUT -p tcp -d <iscsi_target_ip> --dport 3260 -j DROP`
   - `iptables -I INPUT  -p tcp -s <iscsi_target_ip> --dport 3260 -j DROP`
3. Kesinti sırasında cluster davranışını izler
4. İSCSI kurtarmasını bekler (30 + 15s)
5. `pcs resource cleanup` çalıştırır
6. Cluster stabilizasyonunu bekler (Failed Resource Actions yok olana kadar, 24 × 5s)
7. Her iki node'da GFS2 mount durumunu doğrular
8. Multipath kurtarma durumunu raporlar

**always bloğu:** iptables kuralları her iki node'dan kaldırılır.

**Kanıtladığı şeyler:**
- Cluster geçici iSCSI yolu kesintisini yönetir
- Multipath bağlantı yeniden kurulunca path recovery gerçekleştirir
- GFS2 mount'ları iSCSI kurtarma sonrası düzelir

---

### T11 — Yük Altında Failover

**Tag:** `test_T11` | **Tür:** Güvenli | **Her zaman çalışır:** Evet

**Ne yapar:**

1. `ha_load_t11` tablosunu oluşturur
2. Arka planda yük üretir: 1000 INSERT işlemi, 30ms aralıklarla (asenkron, 90s timeout):
   - Her INSERT: `repeat('x', 1000)` — 1KB'lık veri
3. 10 saniye bekler (yükün birikmesi için)
4. Yük devam ederken VIP failover tetikler:
   - `pcs resource move vip <hedef>` ile hedef node'u dinamik belirler
5. Failover tamamlanmasını bekler: PostgreSQL hedef node'da `Started` olana kadar (18 × 5s)
6. Yük bitince VIP üzerinden `SELECT count(*) FROM ha_load_t11;` sorgular
7. Kaydedilen kayıt sayısını raporlar

**always bloğu:** Location constraint kaldırılır, test tablosu drop edilir.

**Kanıtladığı şeyler:**
- Cluster PostgreSQL yük altındayken VIP failover gerçekleştirebilir
- Pacemaker yük koşullarında resource geçişini doğru yönetir
- Failover öncesi commit edilen kayıtlar korunur

---

### T12 — Çoklu Ardışık Failover (Dayanıklılık)

**Tag:** `test_T12` | **Tür:** Güvenli | **Her zaman çalışır:** Evet

**Ne yapar:**

`ha_test_t12` tablosunu oluşturur, ardından `tasks/failover_cycle.yml` dosyasını **3 kez** döngüyle çalıştırır (`failover_cycles: 3`).

**Her döngüde (`failover_cycle.yml`):**

1. VIP'in hangi node'da olduğunu belirler
2. `ha_test_t12` tablosuna kayıt ekler: `(cycle, node)` değerleriyle
3. VIP'i diğer node'a taşır: `pcs resource move vip <hedef>`
4. Failover tamamlanmasını bekler (en fazla 60s, 5s aralıklarla)
5. VIP'in hedef node'a geçtiğini doğrular
6. PostgreSQL'in yeni node'da başlamasını bekler (12 × 5s)
7. VIP üzerinden bağlanarak o döngünün kaydının var olduğunu doğrular (`count = 1`)
8. Location constraint kaldırır (`pcs resource clear vip`)
9. Stabilizasyon bekler (min 30s)

Tüm döngüler tamamlandıktan sonra toplam kayıt sayısı döngü sayısına eşit mi doğrular.

**always bloğu:** Location constraint kaldırılır, test tablosu drop edilir.

**Kanıtladığı şeyler:**
- Cluster birden fazla ardışık failover sonrasında bozulmadan çalışmaya devam eder
- Her failover başarıyla tamamlanır
- Her döngüde yazılan veri tüm sonraki failover'lardan sonra da korunur

---

### T13 — Tam Cluster Yeniden Başlatma

**Tag:** `test_T13` | **Tür:** Yıkıcı | **Gereksinim:** `-e run_destructive=true`

**Ne yapar:**

1. Tüm cluster'ı durdurur: `pcs cluster stop --all --wait=120`
2. Her iki node'da corosync'in durduğunu doğrular
3. 10 saniye bekler
4. Tüm cluster'ı başlatır: `pcs cluster start --all --wait=120`
5. Her iki node'un online olmasını ve tüm resource'ların başlamasını bekler (18 × 10s)
6. `pcs resource cleanup` çalıştırır
7. GFS2'nin her iki node'da başladığını doğrular (12 × 10s)
8. PostgreSQL'in başladığını doğrular (12 × 10s)
9. VIP üzerinden PostgreSQL bağlantısı test eder (6 × 5s)

**rescue bloğu:** Hata durumunda `pcs cluster start --all` ile cluster başlatılmaya çalışılır.

**always bloğu:** `pcs resource cleanup` çalışır.

**Kanıtladığı şeyler:**
- Cluster kontrollü tam durdurma ve yeniden başlatma işlemini gerçekleştirebilir
- Tam restart sonrası tüm resource'lar doğru sırayla başlar
- VIP üzerinden PostgreSQL tam restart sonrası erişilebilir durumdadır

---

### T14 — Web Servisi Testi (Nginx + PHP-FPM)

**Tag:** `test_T14` | **Tür:** Güvenli | **Gereksinim:** Yok

Kurulum sürecinde karşılaşılan tüm web servisi sorunlarını kapsayan kapsamlı test.

#### T14.1 — Ön Koşul Kontrolleri

| Kontrol | Açıklama | Başarısızsa |
|---|---|---|
| nginx binary | `/usr/sbin/nginx -v` her node'da çalışıyor mu | `dnf install nginx -y` |
| httpd parametresi | `pcs resource config nginx \| grep httpd=/usr/sbin/nginx` | `pcs resource update nginx httpd=/usr/sbin/nginx` |
| php-pgsql paketi | `rpm -q php-pgsql` her node'da | `dnf install php-pgsql -y` |
| php-pgsql modülü | `php -m \| grep pgsql` her node'da | `pcs resource cleanup php-fpm-clone` |
| Servis maskesi | nginx/php-fpm masked değil mi | `systemctl unmask nginx php-fpm` |

> **Neden `httpd=/usr/sbin/nginx` zorunlu?** `ocf:heartbeat:nginx` agent Pacemaker'ın kısıtlı systemd ortamında `/usr/sbin` PATH'de olmayabilir. Binary yolu verilmezse agent `OCF_ERR_INSTALLED` (not installed) döner.

> **Neden `php-pgsql` ayrı paket?** `php-mysqlnd` MySQL/MariaDB içindir. `pg_connect()` fonksiyonu için `php-pgsql` (veya PDO pgsql) ayrıca kurulmalıdır.

#### T14.2 — Cluster Kaynak Durumları

- `php-fpm-clone` ve `nginx-clone` her iki node'da `Started` doğrulanır
- Başarısız olursa bilinen sorun/çözüm rehberi gösterilir

#### T14.3 — Order Constraint Doğrulama

Üç zorunlu constraint varlığı kontrol edilir:

```
order-gfs2fs-phpfpm     : gfs2_fs-clone → php-fpm-clone
order-postgresql-phpfpm : postgresql    → php-fpm-clone
order-phpfpm-nginx      : php-fpm-clone → nginx-clone
```

Bu constraint'ler php-fpm ve nginx'in **kesinlikle en son başlamasını** garantiler.

#### T14.4 — Process Kontrolleri

Her node'da `pgrep -a nginx` ve `pgrep -a php-fpm` ile süreç varlığı doğrulanır.

#### T14.5–T14.6 — HTTP/PHP/Status Erişim Testleri

| Test | URL | Beklenen |
|---|---|---|
| HTML | `http://VIP/index.html` | HTTP 200, içerik cluster bilgisi |
| PHP | `http://VIP/index.php` | HTTP 200, PHP SAPI: fpm-fcgi |
| Status | `http://VIP/nginx_status` | HTTP 200, Active connections |

#### T14.7–T14.8 — GFS2 Yazma/Çapraz-Node Okuma Testi

**Önemli düzeltme:** Test dosyası `/shared/webfs/html/tmp/` altına yazılır (nginx kullanıcısı yetkisiyle). GFS2 kök dizinine (`/shared/webfs/`) yazma yetkisi yoktur.

```
Node1 → /shared/webfs/html/tmp/.t14_write_test yaz (nginx kullanıcısı)
Node2 → aynı dosyayı oku (çapraz GFS2 erişim doğrulama)
```

#### T14.9 — PHP → PostgreSQL Bağlantı Testi

`php -r` ile `pg_connect()` fonksiyonu üzerinden PostgreSQL bağlantısı test edilir. `php-pgsql` yoksa PDO fallback denenir, o da yoksa açık hata mesajı gösterilir.

#### T14.10 — Cleanup Sonrası Yeniden Başlatma Testi

`pcs resource cleanup nginx-clone` çalıştırılır, 25 saniye beklenir, HTTP 200 doğrulanır. Bu Pacemaker'ın nginx'i sorunsuz yeniden başlatabildiğini kanıtlar.

#### T14.11 — Hata Analizi

`pcs status | grep 'Failed Resource Actions'` çıktısı parse edilir. Başarısız kaynak varsa bilinen 5 sorun için tam çözüm rehberi gösterilir.

**Kanıtladığı şeyler:**
- Nginx binary doğru konumda ve Pacemaker tarafından bulunabiliyor
- PHP-FPM systemd DBus üzerinden başlatılabiliyor (masked değil)
- GFS2 web kök dizinine nginx kullanıcısı yazabiliyor
- Çapraz-node GFS2 okuma çalışıyor
- php-pgsql kurulu ve PostgreSQL bağlantısı kuruluyor
- Tüm web servisi order constraint'leri doğru tanımlı

**Tek test çalıştırma:**
```bash
ansible-playbook ha_cluster_test.yml -t test_T14
```

---

## 7. RAPOR — Son Doğrulama ve Ortam Restore

Tüm testler tamamlandıktan sonra RAPOR play'i `tags: always` ile her zaman çalışır.

### Kapsamlı Temizlik

1. **iptables temizliği:** Tüm test kaynaklı kurallar tüm node'lardan kaldırılır:
   - corosync (UDP/TCP 5405)
   - qdevice (TCP 5403)
   - iSCSI (TCP 3260)
2. **PostgreSQL tablo temizliği:** Tüm test tabloları drop edilir
3. **GFS2 dosya temizliği:** `/shared/webfs/ha_test_t07_*.txt` silinir
4. **Location constraint temizliği:** `pcs resource clear vip`
5. **Son resource cleanup:** `pcs resource cleanup`
6. **Stabilizasyon bekleme:** 20 saniye

### Son Durum Kontrolleri

7. `pcs status` — son cluster durumu
8. `pcs constraint` — son constraint durumu
9. `pcs quorum status` — son quorum durumu
10. Her iki node'da GFS2 mount durumu: `df -h /shared/webfs`
11. Her iki node'da disk kullanımı: `df -h /shared/webfs /var/lib/pgsql`
12. Son PostgreSQL bağlantı testi: `SELECT 'cluster OK' AS durum, now() AS zaman, version() AS versiyon;`

### Konsol Özet Raporu

```
╔══════════════════════════════════════════════════════════════════════╗
║                  HA CLUSTER TEST RAPORU                              ║
╠══════════════════════════════════════════════════════════════════════╣
║  Tarih    : 2026-03-19 15:30:00
║  Cluster  : ha_webpgsql
║  VIP      : 192.168.0.63:5432
╠══════════════════════════════════════════════════════════════════════╣
║  TEST SONUÇLARI:
║
║  T01 Temel Sağlık Kontrolü          ✅ (her zaman çalışır)
║  T02 VIP Failover                   ✅ (her zaman çalışır)
║  T03 Aktif Node Kapatma             ✅ (çalıştı) / ⏭ (atlandı)
║  ...
╠══════════════════════════════════════════════════════════════════════╣
║  SON CLUSTER DURUMU:
║  [pcs status çıktısı]
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 8. Markdown Rapor Dosyası

Test tamamlandıktan sonra aşağıdaki konuma detaylı bir Markdown rapor dosyası yazılır:

```
/tmp/ha_cluster_test/ha_cluster_test_report_<ZAMAN_DAMGASI>.md
```

### Rapor İçeriği

| Bölüm | İçerik |
|---|---|
| **Genel Bilgiler** | Tarih, cluster adı, node'lar, VIP, mount noktaları, test modu |
| **Sistem Bilgileri** | Kernel versiyonu, OS, pcs/corosync/psql versiyonları |
| **Test Sonuçları Özeti** | Her test için ✅/⏭ durumu tablo formatında |
| **pcs status** | Son cluster durumu tam çıktı |
| **Quorum Durumu** | `corosync-quorumtool -s` tam çıktı |
| **Corosync Ring** | `corosync-cfgtool -s` ring bağlantı durumu |
| **Constraint'ler** | `pcs constraint show --full` tam çıktı |
| **iSCSI Session'ları** | `iscsiadm -m session` aktif session listesi |
| **Multipath Cihazlar** | `multipath -ll` tam çıktı |
| **LVM Lock Durumu** | `lvmlockctl --dump` tam çıktı |
| **Disk / Mount Kullanımı** | Her iki node'da `df -h` çıktısı |
| **PostgreSQL** | VIP bağlantı durumu ve versiyon |
| **Resource Failcount** | `pcs resource failcount show` tam çıktı |
| **Test Detayları** | Her testin ne yaptığının açıklaması |

### Raporu İndirme

```bash
# Cluster node'undan yerel makineye kopyala
scp root@192.168.0.61:/tmp/ha_cluster_test/ha_cluster_test_report_*.md ./ha_cluster_raporu.md

# Veya tüm rapor dizinini kopyala
scp -r root@192.168.0.61:/tmp/ha_cluster_test/ ./cluster_test_raporlari/
```

---

## 9. Güvenlik ve Temizlik Mekanizması

### block / rescue / always Yapısı

T02, T03, T04, T05, T06, T07, T09, T10, T11, T12, T13 testlerinin tamamı `block/rescue/always` ile sarılmıştır:

```yaml
block:
  # Test adımları
rescue:
  # Hata durumunda güvenli çıkış:
  #   - iptables kurallarını kaldır
  #   - Durdurulmuş node'u yeniden başlat
  #   - fail: ile hata mesajı ver
always:
  # Her durumda (başarılı veya başarısız) çalışır:
  #   - iptables kurallarını kaldır
  #   - Durdurulmuş node'u yeniden başlat
  #   - pcs resource cleanup
  #   - Test tablolarını drop et
  #   - pcs resource clear vip
```

Bu yapı sayesinde test yarıda kesilse bile iptables kuralları temizlenir, durdurulmuş node'lar yeniden başlatılır ve test tabloları silinir. **Cluster her koşulda kullanılabilir durumda kalır.**

### Otomatik Ön Temizlik

PREP play'i her test çalışması öncesinde önceki çalışmalardan kalan artıkları temizler:

- Tüm test iptables kuralları tüm node'lardan silinir
- Tüm test tabloları drop edilir
- VIP location constraint'leri temizlenir
- `pcs resource cleanup` çalıştırılır
- 15 saniye stabilizasyon beklenir

**Test playbook'u art arda birden fazla kez güvenle çalıştırılabilir.**

### Temizlenen Kaynaklar Özeti

| Kaynak | Nerede Temizlenir |
|---|---|
| iptables kuralları (5405, 5403, 3260) | PREP + Her test `always` bloğu + RAPOR |
| VIP location constraint'leri | Her test `always` bloğu + RAPOR |
| PostgreSQL test tabloları | Her test `always` bloğu + RAPOR |
| GFS2 test dosyaları (`ha_test_t07_*.txt`) | T07 `always` bloğu + RAPOR |
| Resource failure counter'ları | Her test + RAPOR son cleanup |
| Durdurulmuş node'lar | T03 ve T09 `always` blokları |

---

## 10. Test Hatalarını Giderme

### T01 Hataları

#### `Corosync ring hatası`
```bash
corosync-cfgtool -s
# disconnected veya lost girişlerine bakın

journalctl -u corosync -n 50

# Cluster ağı erişilebilir mi?
ping -I enp2s0 172.16.16.62
```

#### `Quorum device bağlı değil`
```bash
pcs quorum device status
systemctl status corosync-qdevice

# Qdevice yeniden ekle
pcs quorum device remove
pcs quorum device add model net \
  host=clstr-qrm01.lab.akyuz.tech \
  algorithm=ffsplit port=5403
systemctl enable --now corosync-qdevice
```

#### `Başarısız veya durmuş resource var`
```bash
pcs status
pcs resource cleanup
journalctl -u pacemaker -n 50
```

#### `PostgreSQL bağlantı hatası (PREP aşamasında)`
```bash
# pgsql_password doğru mu?
PGPASSWORD="hosts.yml_parola" psql -h 192.168.0.63 -U postgres -c "SELECT 1;"

# PostgreSQL çalışıyor mu?
pcs resource status postgresql

# Bağlantı testi
psql -h 192.168.0.63 -p 5432 -U postgres
# "Password for user postgres:" geliyorsa parola yanlış
```

---

### T02 / T11 / T12 Hataları

#### `VIP timeout içinde taşınamadı`
```bash
# Engelleyici constraint var mı?
pcs constraint

# Resource durumu
pcs resource status vip

# Constraint kaldır ve manuel dene
pcs resource clear vip
pcs resource move vip clstr02.lab.akyuz.tech
```

#### `Failover sonrası PostgreSQL erişilemiyor`
```bash
# PostgreSQL ve VIP aynı node'da mı?
pcs resource status postgresql
pcs resource status vip

# Colocation constraint eksikse ekle
pcs constraint colocation add postgresql with pgsql_fs INFINITY
pcs resource cleanup
```

---

### T03 Hataları

#### `Failover tamamlanamadı`
```bash
# Pasif node'dan kontrol et
pcs status

# Cleanup
pcs resource cleanup

# Gerekirse node'u zorla başlat
pcs cluster start <durdurulmuş_node>
```

#### `Veri tutarlılığı doğrulaması başarısız`
```bash
# Direkt sorgu
psql -h 192.168.0.63 -U postgres \
  -c "SELECT * FROM ha_test_t03;"
```

---

### T06 Hataları

#### `PostgreSQL kill -9 sonrası yeniden başlatılamadı`
```bash
# Resource durumu
pcs resource status postgresql

# Failcount kontrol (< 3 olmalı)
pcs resource failcount show postgresql

# Cleanup ve retry
pcs resource cleanup postgresql
```

#### `Postmaster PID tespit edilemedi`
```bash
# PostgreSQL çalışıyor mu?
pcs resource status postgresql

# Process bul
pgrep -a postgres
ps aux | grep postgres
```

---

### T09 Hataları

#### `IPMI erişilemiyor`
```bash
# Manuel test (aktif node'dan çalıştır)
ipmitool -I lanplus \
  -H <ipmi_ip> -p <ipmi_port> \
  -U <ipmi_user> -P <ipmi_pass> \
  chassis power status

# hosts.yml: ipmi_ip, ipmi_port, ipmi_user, ipmi_password kontrol et
```

#### `async_status timeout — fence işlemi tamamlanamadı`
```bash
# Aktif node'dan güç durumunu kontrol et
ipmitool -I lanplus \
  -H <ipmi_ip> -p <ipmi_port> \
  -U <ipmi_user> -P <ipmi_pass> \
  chassis power status

# IPMI'ya reboot komutu gönder
ipmitool -I lanplus ... chassis power cycle

# Cluster durumunu kontrol et
pcs status
```

#### `Fence edilen node cluster'a geri dönmedi`
```bash
# Node boot etti mi? SSH erişimi var mı?
ssh root@<fence_target_ip>

# Cluster servisini başlat
pcs cluster start <node>

# Veya IPMI ile zorla aç
ipmitool -I lanplus ... chassis power on

# Boot sonrası cluster start
pcs cluster start <node>
pcs resource cleanup
```

#### `OFFLINE penceresi görülmedi (normal durum)`

Bu bir hata değildir. `--reboot` komutu ile fence edilen node hızlı ortamlarda (VM, IPMI simülatör) 5 saniyelik polling penceresi içinde kapanıp açılabilir ve `pcs status` çıktısında hiç `OFFLINE` görünmeden geri dönebilir. Test bu durumu doğru yönetir — fence başarısı `async_status` ile ayrıca doğrulanmaktadır.

---

### T14 Hataları — Web Servisi

#### `nginx start returned 'not installed'`
```bash
# Sorun: ocf:heartbeat:nginx binary'yi PATH'de bulamıyor
# Çözüm:
pcs resource update nginx httpd=/usr/sbin/nginx
pcs resource cleanup nginx-clone

# Doğrula:
pcs resource config nginx | grep httpd
```

#### `php-fpm 'Unable to invoke systemd DBus method'`
```bash
# Sorun: php-fpm servisi masked durumda — DBus ile başlatılamaz
# Çözüm (her iki node'da):
systemctl unmask php-fpm
systemctl unmask nginx    # nginx de maskeli olabilir

# Ardından:
pcs resource cleanup php-fpm-clone
pcs resource cleanup nginx-clone
```

#### `GFS2 Yazma Testi: Başarısız (index.php'de)`
```bash
# Sorun: nginx kullanıcısının /shared/webfs/ kök dizinine yazma izni yok
# Çözüm: tmp dizini oluştur ve sahipliği düzelt
mkdir -p /shared/webfs/html/tmp
chown -R nginx:nginx /shared/webfs/html
chmod 755 /shared/webfs/html/tmp

# SELinux context düzelt (gerekirse):
restorecon -Rv /shared/webfs/html
setsebool -P httpd_use_nfs 1
```

#### `pg_connect fonksiyonu yok (index.php'de)`
```bash
# Sorun: php-pgsql paketi kurulu değil
# php-mysqlnd ≠ php-pgsql, PostgreSQL için ayrı paket gerekir
# Çözüm (her iki node'da):
dnf install php-pgsql -y

# php-fpm yeniden başlat (extension yüklenmesi için):
pcs resource cleanup php-fpm-clone

# Doğrula:
php -m | grep pgsql
```

#### `Eksik web servisi order constraint'leri`
```bash
# Kontrol:
pcs constraint show --full | grep -E "phpfpm|nginx"

# Eksik constraint'leri ekle:
pcs constraint order start gfs2_fs-clone then php-fpm-clone id=order-gfs2fs-phpfpm
pcs constraint order start postgresql then php-fpm-clone id=order-postgresql-phpfpm
pcs constraint order start php-fpm-clone then nginx-clone id=order-phpfpm-nginx
```

#### `nginx veya php-fpm Stopped kaldı (hiç başlamadı)`
```bash
# Adım adım teşhis:
pcs resource debug-start php-fpm    # Hata mesajını görmek için
pcs resource debug-start nginx      # Hata mesajını görmek için

# Pacemaker loglarına bak:
journalctl -u pacemaker -n 100 | grep -E "nginx|php-fpm"

# OCF agent debug:
OCF_ROOT=/usr/lib/ocf /usr/lib/ocf/resource.d/heartbeat/nginx start
```

---

### Genel Temizlik Komutları

Herhangi bir test başarısız olduktan sonra cluster'ı temiz hale getirmek için:

```bash
# iptables kurallarını manuel kaldır (her iki node'da)
iptables -D INPUT  -p udp --dport 5405 -j DROP 2>/dev/null
iptables -D OUTPUT -p udp --dport 5405 -j DROP 2>/dev/null
iptables -D INPUT  -p tcp --dport 5405 -j DROP 2>/dev/null
iptables -D OUTPUT -p tcp --dport 5405 -j DROP 2>/dev/null
iptables -D OUTPUT -p tcp --dport 5403 -j DROP 2>/dev/null
iptables -D INPUT  -p tcp --dport 5403 -j DROP 2>/dev/null
iptables -D OUTPUT -p tcp -d 172.16.16.248 --dport 3260 -j DROP 2>/dev/null
iptables -D INPUT  -p tcp -s 172.16.16.248 --dport 3260 -j DROP 2>/dev/null

# VIP location constraint kaldır
pcs resource clear vip
pcs constraint | grep vip

# Test tablolarını temizle
psql -h 192.168.0.63 -U postgres -c \
  "DROP TABLE IF EXISTS ha_test_t02, ha_test_t03, ha_test_t06,
   ha_load_t11, ha_test_t12 CASCADE;"

# Resource cleanup
pcs resource cleanup

# GFS2 test dosyaları
rm -f /shared/webfs/ha_test_t07_*.txt
rm -f /shared/webfs/html/tmp/.t14_write_test
```

> Temizlik komutlarını tek tek çalıştırmak yerine test playbook'unu yeniden başlatmak da yeterlidir — PREP play'i her durumda temizliği otomatik yapar.

---

*Test playbook versiyonu: son güncelleme 24 Mart 2026 · RHEL 9.7 · Pacemaker 2.1.x · pcs 0.11.x · PostgreSQL 18 · Nginx 1.20.x · PHP 8.3.x · php-pgsql zorunlu · Ansible 2.14.x*
