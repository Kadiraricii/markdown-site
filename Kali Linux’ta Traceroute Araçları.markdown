# Kali Linux’ta Traceroute Araçlarının Kapsamlı Analizi

## 1. Giriş
Traceroute araçları, Kali Linux’ta ağ tanılama için kritik öneme sahip yardımcı programlardır ve IP ağlarında bir kaynaktan hedefe giden paketlerin izlediği yolu haritalandırmak için kullanılır. Artan Yaşam Süresi (TTL) değerleriyle paketler göndererek, bu araçlar her bir sıçramayı (yönlendiriciyi) tanımlar ve geçiş gecikmelerini ölçer, böylece ağ sorunlarını giderme, performans analizi ve keşif faaliyetlerine yardımcı olur. Kali Linux’un traceroute paketi, **traceroute.db**, **traceroute6.db**, **tcptraceroute.db**, **traceproto.db**, **traceroute-nanog** ve **lft.db** gibi birden fazla varyant içerir ve her biri IPv6 tanılama veya güvenlik duvarlarını aşma gibi özel kullanım senaryoları için tasarlanmıştır.

Bu belge, bu araçların kapsamlı, doğru ve profesyonel bir analizini sunar; resmi kaynaklardan alınan bilgileri birleştirir, daha önce tespit edilen hataları (ör. traceroute6.db’nin spekülatif tanımı) düzeltir ve man sayfalarındaki tüm parametreleri içerir. Teknik rapor veya sunum olarak kullanılmak üzere yapılandırılmıştır ve her araç için **Genel Bakış**, **Parametreler**, **Kullanım Örnekleri**, **Avantajlar** ve **Limitasyonlar** bölümleriyle düzenlenmiştir.

**Kaynaklar**:
- [Kali Linux Araçları - Traceroute](https://www.kali.org/tools/traceroute/)
- [Linux man sayfası - traceroute](https://linux.die.net/man/8/traceroute)
- [tcptraceroute man sayfası](https://linux.die.net/man/1/tcptraceroute)
- [LFT - Katman Dört Traceroute](https://pwhois.org/lft/)
- [TraceProto Ana Sayfası](https://traceproto.sourceforge.net/)
- [traceroute-nanog man sayfası](http://man.he.net/man8/traceroute-nanog)
- [traceroute6 man sayfası](https://linux.die.net/man/8/traceroute6)
- [Debian Paket Detayları - traceroute](https://packages.debian.org/buster/traceroute)

## 2. Kurulum
Traceroute paketi Kali Linux’ta her zaman önceden yüklü olmayabilir, ancak aşağıdaki komutla kolayca kurulabilir:

```bash
sudo apt install traceroute
```

Bu, `/usr/bin/traceroute.db`, `/usr/bin/traceroute6.db`, `/usr/sbin/tcptraceroute.db`, `/usr/bin/traceproto.db`, `/usr/bin/traceroute-nanog` ve `/usr/bin/lft.db` gibi ikili dosyaları kurar. Paket yaklaşık 160 KB yer kaplar ve **libc6** bağımlılığına sahiptir.

## 3. Traceroute.db
### Genel Bakış
**traceroute.db**, Kali Linux’taki standart traceroute uygulamasıdır ve Van Jacobson’ın tasarımına dayanır, ek geliştirmelerle zenginleştirilmiştir. UDP, ICMP, TCP ve DCCP problarını destekler ve ağ tanılama ile keşif için kapsamlı özelleştirme seçenekleri sunar.

### Parametreler
Aşağıdaki tablo, [traceroute man sayfası](https://linux.die.net/man/8/traceroute) temel alınarak tüm parametreleri listeler:

| **Seçenek** | **Açıklama** | **Varsayılan** |
|-------------|-------------------------------------------------------------------------------------------------|----------------|
| `-4`, `-6` | IPv4 veya IPv6 izlemeyi zorlar. | Otomatik seçim |
| `-I` | ICMP ECHO problarını kullanır. | Yok |
| `-T` | TCP SYN problarını kullanır (varsayılan port 80). | Yok |
| `-U` | UDP problarını kullanır (varsayılan port 53). | Varsayılan yöntem |
| `-D` | DCCP İstek paketlerini kullanır (varsayılan port 33434). | Yok |
| `-f first_ttl` | İlk TTL’yi ayarlar. | 1 |
| `-m max_ttl` | Maksimum sıçrama sayısını ayarlar. | 30 |
| `-q nqueries` | Her sıçrama için probe sayısı. | 3 |
| `-N squeries` | Eşzamanlı probe sayısı. | 16 |
| `-w waittime` | Yanıt için bekleme süresi (saniye). | 5.0 |
| `-z sendwait` | Problar arası minimum zaman aralığı (10’dan büyükse ms, değilse saniye; ondalıklı değer kabul edilir). | 0 |
| `-n` | Ana bilgisayar adı çözümlemesini devre dışı bırakır. | Yok |
| `-p port` | Hedef portu ayarlar (UDP: 33434, TCP: 80, ICMP: sıra numarası). | Protokole bağlı |
| `-t tos` | IPv4 için Hizmet Türü (TOS) ayarlar (ör. düşük gecikme için 16). | Yok |
| `-s source_addr` | Kaynak IP adresini belirtir. | Çıkış arayüzü |
| `-i interface` | Ağ arayüzünü belirtir. | Yönlendirme tablosu |
| `-g gateway` | Belirtilen ağ geçitleri üzerinden kaynak yönlendirme ekler (virgülle ayrılmış; IPv6: `num,addr,addr...`). | Yok |
| `-r` | Yönlendirme tablosunu atlar, doğrudan bağlı ağdaki bir ana bilgisayara gönderir. | Yok |
| `-e` | ICMP uzantılarını gösterir (RFC 4884), ör. MPLS etiketleri (RFC 4950). | Yok |
| `-F` | Parçalama yapmama bayrağını ayarlar (Yol MTU keşfi için). | Yok |
| `--mtu` | Yol boyunca MTU’yu keşfeder (`-F -N 1` içerir). | Yok |
| `-A` | AS yolu aramalarını gerçekleştirir. | Yok |

### Kullanım Örnekleri
1. **Temel İzleme**: `traceroute google.com`  
   UDP problarıyla (port 33434) yolu izler, IP’leri ve gidiş-dönüş sürelerini gösterir.
2. **TCP İzleme**: `traceroute -T google.com`  
   TCP SYN problarıyla (port 80) izler, UDP’yi engelleyen güvenlik duvarlarını aşar.
3. **Probe Aralığı**: `traceroute -z 0.5 google.com`  
   Problar arasında 500ms bekler, ağ yükünü azaltır.
4. **Doğrudan Yönlendirme**: `traceroute -r 192.168.1.1`  
   Yönlendirme tablolarını atlayarak yerel bir ana bilgisayara doğrudan paket gönderir.
5. **Yol MTU Keşfi**: `traceroute -F google.com`  
   Paket parçalanmasını önler, MTU analizi için kullanılır.
6. **Kaynak Yönlendirme**: `traceroute -g 192.168.1.1 google.com`  
   Paketleri belirtilen bir ağ geçidi üzerinden yönlendirir.

### Avantajlar
- UDP, ICMP, TCP ve DCCP gibi birden fazla protokolü destekler, çok yönlüdür.
- TTL, probe ve kaynak yönlendirme gibi seçeneklerle yüksek düzeyde özelleştirilebilir.
- Gecikme, MPLS etiketleri ve AS numaraları gibi ayrıntılı çıktılar sağlar.

### Limitasyonlar
- Güvenlik duvarları UDP/ICMP problarını engelleyebilir, eksik izlemelere neden olabilir (çıktıda yıldız işaretleri).
- Kaynak yönlendirme (-g), güvenlik nedeniyle yönlendiriciler tarafından sıklıkla devre dışı bırakılır.
- DCCP (-D), modern ağlarda nadiren desteklenir.

## 4. Traceroute6.db
### Genel Bakış
**traceroute6.db**, IPv6 ağları için özel olarak tasarlanmış traceroute aracıdır ve `traceroute -6` ile eşdeğerdir. traceroute.db ile aynı kod tabanını paylaşır, ancak IPv6 ağları için yapılandırılmıştır; daha önce spekülatif olarak "wrapper" olarak tanımlanması düzeltilmiştir.

### Parametreler
[traceroute6 man sayfası](https://linux.die.net/man/8/traceroute6) temel alınarak, traceroute.db seçeneklerinin çoğu desteklenir, IPv6’ya özgü uyarlamalarla:

| **Seçenek** | **Açıklama** | **Varsayılan** |
|-------------|-------------------------------------------------------------------------------------------------|----------------|
| `-I` | ICMPv6 ECHO problarını kullanır. | Yok |
| `-U` | UDP problarını kullanır. | Varsayılan yöntem |
| `-T` | TCP SYN problarını kullanır (varsayılan port 80). | Yok |
| `-f first_ttl` | İlk TTL’yi ayarlar. | 1 |
| `-m max_ttl` | Maksimum sıçrama sayısını ayarlar. | 30 |
| `-q nqueries` | Her sıçrama için probe sayısı. | 3 |
| `-w waittime` | Yanıt için bekleme süresi (saniye). | 5.0 |
| `-z sendwait` | Problar arası minimum zaman aralığı (10’dan büyükse ms, değilse saniye). | 0 |
| `-n` | Ana bilgisayar adı çözümlemesini devre dışı bırakır. | Yok |
| `-p port` | Hedef portu ayarlar (UDP: 33434, TCP: 80). | Protokole bağlı |
| `-s source_addr` | Kaynak IPv6 adresini belirtir. | Çıkış arayüzü |
| `-i interface` | Ağ arayüzünü belirtir. | Yönlendirme tablosu |
| `-g gateway` | IPv6 Yönlendirme başlıklarıyla kaynak yönlendirme ekler (format: `num,addr,addr...`; RFC 5095). | Yok |
| `-A` | AS yolu aramalarını gerçekleştirir. | Yok |

**Not**: IPv6 uzantı başlıkları (ör. Hop-by-Hop seçenekleri), `-g` (Yönlendirme başlıkları) dışında doğrudan manipüle edilmez.

### Kullanım Örnekleri
1. **Temel IPv6 İzleme**: `traceroute6.db 2001:4860:4860::8888`  
   Google’ın IPv6 DNS sunucusuna UDP problarıyla izler.
2. **TCP İzleme**: `traceroute6.db -T 2001:4860:4860::8888`  
   IPv6 ağları için TCP SYN problarını kullanır.
3. **Kaynak Yönlendirme**: `traceroute6.db -g "2,2001:db8::1" 2001:4860:4860::8888`  
   Belirtilen IPv6 ağ geçidi üzerinden yönlendirir.
4. **DNS Çözümlemesi Olmadan**: `traceroute6.db -n 2001:4860:4860::8888`  
   Yalnızca IP adreslerini gösterir, işlemi hızlandırır.

### Avantajlar
- IPv6 ağ tanıları için vazgeçilmezdir, IPv6 yaygınlığı arttıkça kritik hale gelir.
- Kaynak yönlendirme ve AS aramalarını destekler, gelişmiş tanılar için uygundur.
- traceroute.db’nin güçlü özellik setini paylaşır.

### Limitasyonlar
- Yalnızca IPv6 ağlarıyla sınırlıdır; IPv4 için etkisizdir.
- Güvenlik duvarları IPv6 problarını engelleyebilir, IPv4’e benzer şekilde.
- Gelişmiş IPv6 özellikleri (ör. Hop-by-Hop seçenekleri) hakkında sınırlı dokümantasyon.

## 5. Tcptraceroute.db
### Genel Bakış
**tcptraceroute.db**, TCP SYN paketleriyle yol izleme yapar ve UDP veya ICMP’yi engelleyen güvenlik duvarlarını aşmak için idealdir. Tam TCP bağlantılarından kaçınmak için yarı açık tarama kullanır.

### Parametreler
[tcptraceroute man sayfası](https://linux.die.net/man/1/tcptraceroute) temel alınarak:

| **Seçenek** | **Açıklama** | **Varsayılan** |
|-------------|-----------------------------------------------------------------------------|----------------|
| `-n` | Sayısal çıktı, RFC1918 dışı adresler için DNS çözümlemesi yapmaz. | Yok |
| `-f first_ttl` | İlk TTL’yi ayarlar. | 1 |
| `-m max_ttl` | Maksimum TTL’yi ayarlar. | 30 |
| `-q nqueries` | Her sıçrama için probe sayısı. | 3 |
| `-w waittime` | Her probe için zaman aşımı (saniye). | 3 |
| `-S` | TCP SYN bayrağını ayarlar. | Varsayılan |
| `-A` | SYN yerine TCP ACK bayrağını ayarlar. | Yok |
| `-F` | Parçalama yapmama bayrağını ayarlar. | Yok |
| `-l length` | Toplam TCP paketi uzunluğunu ayarlar. | Belirtilmemiş |
| `-t tos` | Hizmet Türü (TOS; ör. düşük gecikme için 16). | Ayarlanmamış |
| `-E` | TCP SYN paketlerini Açık Tıkanıklık Bildirimi (ECN; RFC 2481) ile gönderir. | Belirtilmemiş |
| `-p port` | Hedef portu ayarlar. | 80 |
| `-s source_addr` | Kaynak IP adresini belirtir. | Çıkış arayüzü |

### Kullanım Örnekleri
1. **Temel TCP İzleme**: `tcptraceroute.db google.com 80`  
   google.com’un web sunucusuna (port 80) izleme yapar.
2. **Özel Paket Uzunluğu**: `tcptraceroute.db -l 100 google.com 80`  
   Paket uzunluğunu 100 bayta ayarlar.
3. **TOS Ayarı**: `tcptraceroute.db -t 16 google.com 80`  
   Düşük gecikmeyi önceliklendirir.
4. **ECN İzleme**: `tcptraceroute.db -E google.com 80`  
   Tıkanıklık geri bildirimi için ECN’yi etkinleştirir.

### Avantajlar
- TCP kullanarak UDP/ICMP’yi engelleyen güvenlik duvarlarını aşar.
- TOS ve ECN’yi destekler, gelişmiş ağ analizi için uygundur.
- Güvenlik duvarlı ortamlarda güvenilirdir.

### Limitasyonlar
- Hedefte açık bir TCP portu gerektirir.
- TCP dışı senaryolar için traceroute.db’den daha az çok yönlüdür.
- ECN desteği, ağ uyumluluğuna bağlıdır.

## 6. Traceproto.db
### Genel Bakış
**traceproto.db**, kullanıcı tarafından belirtilen protokolleri (TCP, UDP, ICMP) ve portları destekleyen bir traceroute alternatifidir ve güvenlik duvarlı ortamlarda belirli hizmetleri test etmek için tasarlanmıştır.

### Parametreler
[Traceproto man sayfası](https://linux.die.net/man/8/traceproto) temel alınarak:

| **Seçenek** | **Açıklama** | **Varsayılan** |
|-------------|-------------------------------------------------------------------------------------------------|----------------|
| `-p protocol` | Protokolü belirtir (tcp, udp, icmp). | udp |
| `-d dst_port` | Hedef portu ayarlar. | Protokole bağlı |
| `-s source_port` | Kaynak portu ayarlar (artış desenleriyle kullanıldığında minimum port). | 10240 |
| `-S max_source_port` | Artan portlar için maksimum kaynak portu ayarlar. | Yok |
| `-m min_ttl` | Minimum TTL’yi ayarlar. | 1 |
| `-M max_ttl` | Maksimum TTL’yi ayarlar. | 30 |
| `-W interval` | Problar arası gecikmeyi ayarlar (milisaniye). | 100 |
| `-i pattern` | Port artış desenini ayarlar (s=kaynak, d=hedef, S=kaynak sonra hedef, D=hedef sonra kaynak). | Yok |
| `-q nqueries` | Her sıçrama için probe sayısı. | 3 |
| `-n` | Ana bilgisayar adı çözümlemesini devre dışı bırakır. | Yok |

### Kullanım Örnekleri
1. **TCP İzleme**: `traceproto.db -p tcp -d 80 google.com`  
   TCP ile port 80’e izleme yapar.
2. **Kaynak Portu**: `traceproto.db -s 5000 -p tcp google.com`  
   Kaynak portunu 5000’e ayarlar.
3. **Probe Aralığı**: `traceproto.db -W 200 google.com`  
   Problar arasında 200ms ayarlar.
4. **ICMP İzleme**: `traceproto.db -p icmp google.com`  
   ICMP problarıyla izleme yapar.

### Avantajlar
- Belirli protokoller ve portlar için yüksek düzeyde özelleştirilebilir.
- Güvenlik duvarlı ortamlarda ağ hizmetlerini test etmek için etkilidir.
- Özelleştirilebilir çıktı stilleriyle betik oluşturmayı destekler.

### Limitasyonlar
- Hedef hizmetler hakkında bilgi gerektiren karmaşık yapılandırma.
- Standart traceroute’a kıyasla sınırlı yaygınlık.
- Port artış desenleri (-i), yanıt yorumlamasını karmaşıklaştırabilir.

## 7. Traceroute-nanog
### Genel Bakış
**traceroute-nanog** (trACESroute olarak da bilinir), AS numarası aramaları, TOS desteği ve mikro saniye zaman damgaları gibi özelliklerle geliştirilmiş bir traceroute uygulamasıdır ve gelişmiş ağ analizi için idealdir.

### Parametreler
[traceroute-nanog man sayfası](http://man.he.net/man8/traceroute-nanog) temel alınarak:

| **Seçenek** | **Açıklama** | **Varsayılan** |
|-------------|-------------------------------------------------------------------------------------------------|----------------|
| `-I proto` | IP protokolünü belirtir (ör. udp, icmp; paralel izleme için UDP gerekir). | udp |
| `-A` | Her sıçrama için AS numarası aramasını etkinleştirir (ör. whois.ra.net üzerinden). | Yok |
| `-P` | Daha hızlı izleme için paralel probeyi etkinleştirir. | Yok |
| `-f first_ttl` | İlk TTL’yi ayarlar. | 1 |
| `-m max_ttl` | Maksimum TTL’yi ayarlar. | 30 |
| `-q nqueries` | Her sıçrama için probe sayısı. | 3 |
| `-w waittime` | Yanıt için bekleme süresi (saniye). | 5.0 |
| `-n` | Ana bilgisayar adı çözümlemesini devre dışı bırakır. | Yok |
| `-t tos` | Hizmet Türü (TOS; ör. düşük gecikme için 16). | Yok |
| `-F` | Parçalama yapmama bayrağını ayarlar. | Yok |
| `--mtu` | Yol boyunca MTU’yu keşfeder. | Yok |

**Not**: Daha önce önerilen `-B` (BGP tabanlı AS yolu kontrolü) seçeneği man sayfasında bulunmamaktadır.

### Kullanım Örnekleri
1. **Temel İzleme**: `traceroute-nanog google.com`  
   Geliştirilmiş özelliklerle UDP problarıyla izler.
2. **AS Araması**: `traceroute-nanog -A google.com`  
   Her sıçrama için AS numaralarını gösterir.
3. **ICMP İzleme**: `traceroute-nanog -I icmp google.com`  
   ICMP problarını kullanır.
4. **Paralel Probing**: `traceroute-nanog -P google.com`  
   Paralel problarla izlemeyi hızlandırır.

### Avantajlar
- AS aramaları, MTU keşfi ve yüksek hassasiyetli zaman damgalarıyla zengin özellik seti.
- Paralel izleme (-P), izlemeyi hızlandırır.
- Büyük ölçekli ağ analizi için idealdir.

### Limitasyonlar
- Paralel izleme, ICMP hız sınırlaması nedeniyle paket kaybına yol açabilir.
- AS arama doğruluğu, harici kayıt veritabanlarına bağlıdır.
- Basit tanılar için gereğinden fazla karmaşık olabilir.

## 8. Lft.db
### Genel Bakış
**lft.db** (Katman Dört Traceroute), yük dengeleyiciler ve güvenlik duvarları içeren karmaşık ağlar için tasarlanmış hızlı, çok protokollü bir traceroute aracıdır. TCP, UDP, ICMP ve RFC 1393 izleme yöntemlerini destekler.

### Parametreler
[LFT man sayfası](https://linux.die.net/man/8/lft) temel alınarak:

| **Seçenek** | **Açıklama** | **Varsayılan** |
|-------------|-------------------------------------------------------------------------------------------------|----------------|
| `-p dport` | Hedef portu ayarlar. | Protokole bağlı |
| `-s source_port` | Kaynak TCP portunu ayarlar. | 53 |
| `-b` | TCP temel izlemeyi etkinleştirir (sıra numarasına bağımlı olmadan, NAT uyumlu ancak daha yavaş). | Yok |
| `-H` | Maksimum TTL’yi ayarlar. | 30 |
| `-q nqueries` | Her sıçrama için probe sayısı. | 3 |
| `-w waittime` | Yanıt için bekleme süresi (saniye). | 2.0 |
| `-n` | Ana bilgisayar adı çözümlemesini devre dışı bırakır. | Yok |
| `-E` | Genişletilmiş modu etkinleştirir (ayrıntılı çıktı). | Yok |
| `-A` | AS numarası aramasını etkinleştirir. | Yok |

**Not**: Daha önce önerilen `-b` (bant genişliği sınırlaması) ve `-H` (özel TCP başlıkları) tanımları, man sayfasına uygun şekilde düzeltilmiştir.

### Kullanım Örnekleri
1. **Temel İzleme**: `lft.db google.com`  
   Varsayılan ayarlarla TCP problarıyla izler.
2. **Kaynak Portu**: `lft.db -s 12345 google.com`  
   Kaynak portunu 12345’e ayarlar.
3. **Temel İzleme**: `lft.db -b google.com`  
   NAT uyumluluğu için TCP temel yöntemini kullanır.
4. **Maksimum TTL**: `lft.db -H 20 google.com`  
   İzlemeyi 20 sıçramayla sınırlandırır.

### Avantajlar
- Standart traceroute’tan daha hızlıdır ve güçlü güvenlik duvarı geçişi sağlar.
- AS aramalarını ve yük dengeleyici algılamayı destekler.
- TCP, UDP ve ICMP gibi esnek protokol desteği sunar.

### Limitasyonlar
- Karmaşık çıktı, yorumlamak için uzmanlık gerektirir.
- Bazı seçenekler (ör. RFC 1393), uyumsuz ağ ekipmanları nedeniyle başarısız olabilir.
- Belirli seçenekler için dokümantasyon tutarsızlıkları.

## 9. Pratik Hususlar
### Ağ Ortamı
Traceroute’un etkinliği, ağ yapılandırmalarına bağlıdır:
- **Güvenlik Duvarları/NAT**: ICMP, UDP veya belirli portları engelleyebilir, çıktıda yıldız işaretleri (*) oluşturur.
- **ICMP Hız Sınırlaması**: Yanıtları geciktirebilir veya engelleyebilir.
- tcptraceroute.db ve lft.db gibi varyantlar bazı sorunları hafifletir, ancak tamamen kusursuz değildir.

### Güvenlik Etkileri
Traceroute, Kali Linux’ta bir keşif aracıdır ve genellikle penetrasyon testlerinde kullanılır. Aşırı kullanım, IDS/IPS sistemleri tarafından şüpheli olarak algılanabilir. Üretim ortamlarında otomatik betikler çalıştırmaktan kaçınılmalı, ağ yükü veya uyarıları önlemek için.

### Çıktı Yorumlama
Traceroute çıktısı şunları içerir:
- **Sıçrama Numarası**: Yönlendiricinin sıralı konumu.
- **IP/Ana Bilgisayar Adı**: Çözümlenen adres (`-n` kullanılmadıkça).
- **Gidiş-Dönüş Süresi**: Her probe için gecikme.
- **Açıklamalar**:
  - `*`: Yanıt yok (genellikle filtreleme nedeniyle).
  - `!H`: Ana bilgisayara ulaşılamıyor.
  - `!F`: Parçalama gerekli.
  - `!X`: İletişim yasak.

## 10. Araçların Karşılaştırması
| **Araç** | **En İyi Kullanım Senaryosu** | **Temel Güç** | **Temel Limitasyon** |
|----------|-------------------------------|----------------|----------------------|
| traceroute.db | Genel amaçlı izleme | Çok yönlü protokol desteği | Güvenlik duvarları tarafından engellenir |
| traceroute6.db | IPv6 tanılama | IPv6’ya özgü izleme | Yalnızca IPv6 ile sınırlı |
| tcptraceroute.db | Güvenlik duvarlı ortamlar | UDP/ICMP filtrelerini aşar | Açık TCP portu gerekir |
| traceproto.db | Belirli protokol/port testi | Yüksek özelleştirme | Karmaşık kurulum |
| traceroute-nanog | Gelişmiş ağ analizi | AS aramaları, hassasiyet | Basit görevler için fazla karmaşık |
| lft.db | Hızlı, karmaşık ağ izleme | Hız, güvenlik duvarı geçişi | Karmaşık çıktı yorumlama |

## 11. Sonuç
Kali Linux’un traceroute araç seti, ağ tanılama ve güvenlik testi için güçlü bir araç koleksiyonu sunar. **traceroute.db**, çok yönlü bir temel sağlar; **traceroute6.db**, **tcptraceroute.db**, **traceproto.db**, **traceroute-nanog** ve **lft.db** ise IPv6 izlemeden güvenlik duvarı geçişine kadar özel ihtiyaçları karşılar. Bu belge, önceki hataları (ör. traceroute6.db’nin "wrapper" iddiası) düzeltir, tüm man sayfası parametrelerini içerir ve etkili kullanım için pratik örnekler sunar. Her aracın güçlü yönlerini ve limitasyonlarını anlayarak, kullanıcılar ağ ortamlarına en uygun varyantı seçebilir, doğru sorun giderme ve keşif sağlayabilir.