# Contiki-NG ELF Binary Analiz Raporu

**Dosya:** `udp-server.z1`  
**Tarih:** 2026-06-06  
**Platform:** Zolertia Z1 (Texas Instruments MSP430F2617)

---

## 1. ELF Başlık Bilgileri

| Alan | Değer |
|------|-------|
| Magic | `7f 45 4c 46 01 01 01 ff` |
| Sınıf | ELF32 |
| Veri Düzeni | 2'ye tümleyen, little-endian |
| Versiyon | 1 (güncel) |
| OS/ABI | Standalone App |
| Tip | EXEC (Çalıştırılabilir Dosya) |
| Mimari | Texas Instruments MSP430 |
| Giriş Noktası | `0x3100` |
| Program Header Sayısı | 5 |
| Bölüm Header Sayısı | 20 |

---

## 2. Bellek Haritası ve Bölüm Analizi

### Çalışma Bölümleri

| Bölüm | Adres | Boyut (hex) | Boyut (byte) | Bayraklar | Açıklama |
|-------|-------|-------------|--------------|-----------|----------|
| `.text` | `0x3100` | `0xA0C6` | 41,158 | AX | Çalıştırılabilir kod |
| `.rodata` | `0xD1C8` | `0x0553` | 1,363 | A | Salt-okunur veri |
| `.data` | `0x1100` | `0x0150` | 336 | WA | Başlatılmış global veri |
| `.bss` | `0x1250` | `0x16E8` | 5,864 | WA | Başlatılmamış global veri |
| `.noinit` | `0x2938` | `0x0002` | 2 | WA | Kalıcı (sıfırlanmayan) veri |
| `.vectors` | `0xFFC0` | `0x0040` | 64 | AX | Kesme vektör tablosu |

### Hata Ayıklama Bölümleri

| Bölüm | Offset | Boyut | Açıklama |
|-------|--------|-------|----------|
| `.debug_info` | `0x00abb8` | 6,950 byte | DWARF hata ayıklama bilgisi |
| `.debug_line` | `0x00d099` | 4,099 byte | Satır numarası bilgisi |
| `.debug_abbrev` | `0x00c6de` | 2,491 byte | Abbreviation tablosu |
| `.debug_loc` | `0x00e77a` | 5,677 byte | Konum ifadeleri |
| `.debug_str` | `0x00e30c` | 1,134 byte | Hata ayıklama string tablosu |
| `.debug_frame` | `0x00e09c` | 624 byte | Frame bilgisi |
| `.debug_ranges` | `0x00fda7` | 264 byte | Adres aralıkları |
| `.symtab` | `0x0102a4` | 18,160 byte | Sembol tablosu (387+ sembol) |

---

## 3. Boyut Özeti

```
text     data     bss      dec      hex      dosya
42,585   336      5,866    48,787   0xBE93   udp-server.z1
```

- **Toplam Flash Kullanımı:** ~43 KB (`.text` + `.rodata`)
- **Toplam RAM Kullanımı:** ~6.2 KB (`.data` + `.bss` + `.noinit`)

---

## 4. Program Header (Segment) Analizi

| Segment | Tip | VirtAddr | Boyut | Bayraklar | İçerik |
|---------|-----|----------|-------|-----------|--------|
| 0 | LOAD | `0x0000302C` | 0xA19A | R E | `.text` |
| 1 | LOAD | `0x0000D1C8` | 0x0553 | R | `.rodata` |
| 2 | LOAD | `0x00001100` | 0x1838 | RW | `.data` + `.bss` |
| 3 | LOAD | `0x00002938` | 0x0002 | RW | `.noinit` |
| 4 | LOAD | `0x0000FFC0` | 0x0040 | R E | `.vectors` |

---

## 5. Tespit Edilen Contiki-NG Süreçleri

Aşağıdaki Protothread/Process nesneleri sembol tablosundan tespit edilmiştir:

| Sembol | Adres | Açıklama |
|--------|-------|----------|
| `accmeter_process` | `0x1100` | İvmeölçer (ADXL345) sürücü süreci |
| `cc2420_process` | `0x110C` | CC2420 radyo yonga sürücüsü |
| `ctimer_process` | `0x1130` | Callback zamanlayıcı süreci |
| `etimer_process` | `0x113C` | Olay zamanlayıcı süreci |
| `sensors_process` | `0x11BE` | Sensör yönetim süreci |
| `simple_udp_process` | `0x11CA` | UDP iletişim süreci |
| `stack_check_process` | `0x11D6` | Stack taşma kontrol süreci |
| `tcpip_process` | `0x11E2` | TCP/IP ağ yığını süreci |
| `udp_server_process` | `0x11EE` | **Ana uygulama: UDP sunucu** |

---

## 6. Ağ Yığını Bileşenleri

| Bileşen | Fonksiyon | Adres |
|---------|-----------|-------|
| **UIP** | Micro-IP işlemcisi | `0xBD1C` (`uip_process`) |
| **6LoWPAN** | IEEE 802.15.4 paket sıkıştırma | `0x8C4E` (`sicslowpan_init`) |
| **RPL** | IPv6 yönlendirme protokolü | `0x6C9C` (rpl_* fonksiyonlar) |
| **CSMA** | Ortam erişim kontrolü | `0x499C` (`csma_output_init`) |
| **UDP** | Veri birimi protokolü | `0xA476` (`udp_new`) |
| **ICMPv6** | Kontrol mesajları | `0xB4F6` (`uip_icmp6_init`) |
| **ND6** | IPv6 komşu keşfi | `0xB79A` (`uip_nd6_init`) |

---

## 7. Donanım Sürücüleri

| Sürücü | Başlatma Fonksiyonu | Adres |
|--------|---------------------|-------|
| CC2420 Radyo | `cc2420_arch_init` | `0x3AD6` |
| I2C Alıcı | `i2c_receiveinit` | `0x554E` |
| I2C Verici | `i2c_transmitinit` | `0x5584` |
| LED | `leds_arch_init` | `0x5632` |
| MSP430 DCO | `msp430_init_dco` | `0x5CC8` |
| MSP430 CPU | `msp430_cpu_init` | `0x5D68` |
| RTIMER | `rtimer_arch_init` | `0x8AC6` |
| SPI | `spi_init` | `0xA282` |
| TMP102 (Sıcaklık) | `tmp102_init` | `0xA72C` |
| UART0 | `uart0_init` | `0xA870` |
| Watchdog | `watchdog_init` | `0xC4D8` |
| XMEM (Harici Flash) | `xmem_init` | `0xC530` |

---

## 8. Başlatma Akışı

Giriş noktası `0x3100` adresindeki `_reset_vector__` fonksiyonundan başlar:

```
_reset_vector__  (0x3100)
  └─> __init_stack      (0x310C)
  └─> __low_level_init  (0x3110)
  └─> main              (0x313E)
        ├─> platform_init_stage_one   (0x6432)
        ├─> platform_init_stage_two   (0x6444)
        ├─> platform_init_stage_three (0x64FE)
        ├─> process_init              (0x66CA)
        ├─> autostart_start           (0x3A3A)  ← autostart_processes (0xD34C)
        └─> [Process scheduler loop]
```

---

## 9. RPL Yönlendirme Analizi

RPL (Routing Protocol for Low-Power and Lossy Networks) tam implementasyonu mevcut:

- `rpl_dag_root_start` (`0x696C`) — DAG kök başlatma
- `rpl_process_dio` (`0x6DE0`) — DODAG bilgi nesnesi işleme
- `rpl_process_dis` (`0x6F7E`) — DODAG bilgi talep işleme
- `rpl_process_dao` (`0x6FA8`) — Hedef yayını işleme
- `rpl_process_dao_ack` (`0x700E`) — DAO onay işleme
- `rpl_timers_init` (`0x87D8`) — Trickle zamanlayıcısı

---

## 10. Güvenlik ve Kalite Notları

| Konu | Durum | Detay |
|------|-------|-------|
| Stack koruma | Aktif | `stack_check_process` çalışıyor |
| Watchdog | Aktif | `watchdog_init` + `watchdog_start` mevcut |
| Debug sembolleri | Mevcut | DWARF bilgisi binary içinde |
| Dış bağımlılık | 1 adet | `gpio_hal_arch_init` (tanımsız sembol `U`) |
| Mimariye özgü bayraklar | Uyarı | `unknown extra flag bits` ELF bayraklarında |

> **Not:** `gpio_hal_arch_init` fonksiyonu tanımsız (`U`) olarak işaretlenmiştir. Bu, link zamanında çözümlenmesi gereken harici bir bağımlılığa işaret eder; ancak EXEC tipli binary'de bu durum beklenmediktir. Bağlantı adımının gözden geçirilmesi önerilir.

---

## 11. Özet

`udp-server.z1`, Contiki-NG işletim sistemi üzerinde çalışan, **Zolertia Z1 (MSP430)** hedef platformuna derlenmiş bir **IEEE 802.15.4 / IPv6 / UDP sunucu** uygulamasıdır. Binary, tam bir IoT ağ yığını (6LoWPAN, RPL, UIP) ve donanım sürücü seti içermektedir. Flash bellek kullanımı ~43 KB, RAM kullanımı ~6.2 KB olup MSP430F2617'nin kapasitesi dahilindedir.
