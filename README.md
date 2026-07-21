# 🤖 El-Ceziri 8-Bit İşlemci Simülasyonu (Execution)
 
Bu proje, kurgusal **8-bit "El-Ceziri" Assembly mimarisini** yazılımsal olarak simüle eden saf Java programlarından oluşur. Bir kaynak dosyada tanımlı Assembly komutlarını satır satır okur; kayıtçı, bellek ve bayrak (flag) durumlarını gerçek bir CPU'nun **fetch → decode → execute** döngüsüne benzer şekilde günceller.
 
## 📁 Repo İçeriği
 
| Dosya | Açıklama |
|---|---|
| `Execution.java` | Ana simülatör — geniş Türkçe açıklama yorumları içerir, `execution` paketinde |
| `Execution3.java` | Aynı mantığın yorumsuz/sade hâli, `execution3` paketinde (fonksiyonel olarak `Execution.java` ile birebir aynı) |
| `program.txt` | Örnek kaynak kod: girilen N sayısına kadar olan değerleri RAM'e yazıp geri okur |
| `Odev3-1.txt` | Örnek program: bölme + çarpma ile doğrulama testi |
| `Odev3-2.txt` | Örnek program: koşullu dallanma (`DK`) ile min/max mantığı |
 
> İki `.java` dosyası da bağımsız `public class` içerdiği için ayrı ayrı derlenip çalıştırılabilir — birbirlerinin varyantıdır, birlikte çalışmazlar.
 
## 🧠 Simüle Edilen Mimari
 
### Kayıtçılar ve Bellek
 
| Bileşen | Java Karşılığı | Açıklama |
|---|---|---|
| Kayıtçılar | `int AX, BX, CX, DX` | 4 adet 8-bit işaretli kayıtçı, değer aralığı `[-128, 127]` |
| RAM | `int[256]` | 256 adreslik ana bellek |
| Bayraklar | `BayrakSifir, BayrakIsaret, BayrakTasma` | Sırasıyla Zero Flag (ZF), Sign Flag (SF), Overflow Flag (OF) |
| Program Sayacı | `int pc` | Çalıştırılacak komutun program listesindeki indeksi |
| Komut Belleği | `List<String[]> program` | Kaynak dosyadan ayrıştırılmış (tokenize edilmiş) komut satırları |
| Etiket Tablosu | `Map<String, Integer> labels` | Etiket adı → satır numarası eşlemesi (dallanma hedefleri için) |
 
### Yürütme Döngüsü
 
`executeProgram()`, klasik CPU kontrol birimini taklit eder:
1. **Fetch:** `program.get(pc)` ile geçerli komut satırı alınır.
2. **Decode:** `switch(komut)` bloğu, ilk token'a (`k[0]`) göre ilgili işleyici fonksiyona yönlendirir.
3. **Execute:** İlgili fonksiyon işlemi yapar ve çoğunlukla kendi içinde `pc++` ile ilerler; dallanma komutları başarılıysa `pc`'yi doğrudan hedef etikete ayarlar.
## 📜 Desteklenen Komut Seti
 
| Komut | Kategori | Anlamı |
|---|---|---|
| `OKU reg` | G/Ç | Kullanıcıdan tam sayı okur, 8-bit'e taşırır |
| `YAZ hedef` | G/Ç | Kayıtçı / bellek / sabit değeri ekrana yazar |
| `ATM hedef kaynak` | Veri Transferi | Kayıtçı ↔ kayıtçı, sabit, `[BX]` (dolaylı) veya `[100]` (doğrudan) bellek adresleri arası atama |
| `TOP / CIK / CRP / BOL` | Aritmetik | Toplama, çıkarma, çarpma, bölme |
| `VE / VEY / DEG` | Lojik | Bitwise AND, OR, NOT (mantıksal değil, bit bazlı) |
| `D etiket` | Dallanma | Koşulsuz atlama |
| `DE / DED etiket` | Dallanma | ZF=1 / ZF=0 ise atla |
| `DK / DB etiket` | Dallanma | SF=1 (negatif) / SF=0 (pozitif) ise atla |
| `DKE / DBE etiket` | Dallanma | SF=1∨ZF=1 / SF=0∨ZF=1 ise atla (küçük-eşit / büyük-eşit) |
| `NOP` | Kontrol | Hiçbir işlem yapmaz, sadece etiket adresi tutar |
| `SON` | Kontrol | Programı sonlandırır, tüm kayıtçı değerlerini ekrana basar |
 
### Adresleme Modları (`ATM` komutu üzerinden)
 
- **Sabit (Immediate):** `ATM AX 50` → AX = 50
- **Kayıtçıdan Kayıtçıya:** `ATM DX AX` → DX = AX
- **Kayıtçı Dolaylı:** `ATM [BX] AX` → RAM[BX] = AX
- **Doğrudan Adresleme:** `ATM [100] AX` → RAM[100] = AX
### Bayrak Kuralları
 
- **Aritmetik işlemler** (`TOP`, `CIK`, `CRP`, `BOL`): Sonuç Java'nın 32-bit `int`'inde hesaplanır; `[-128, 127]` aralığını aşarsa `BayrakTasma = 1` set edilir ve sonuç `(byte)` cast'iyle 8-bit'e indirgenerek kayıtçıya yazılır.
- **Lojik işlemler** (`VE`, `VEY`, `DEG`): Bit bazlı olduklarından taşma oluşmaz, `BayrakTasma` daima `0`'dır.
- Her işlem sonrası `BayrakSifir` (sonuç==0) ve `BayrakIsaret` (sonuç<0) güncellenir.
## 🚀 Örnek Çalıştırma (`program.txt`)
 
Kullanıcıdan alınan `N` değerine kadar sayıları RAM'e yazıp ardından ekrana basan program:
 
```assembly
OKU CX              ; N değerini kullanıcıdan al
ATM DX CX           ; N'yi yedekle
ATM BX 1            ; RAM adres sayacı = 1
ATM AX 0            ; Yazılacak değer = 0
ETIKET1: CIK CX BX  ; CX = CX - BX (döngü koşulu)
DE ETIKET2          ; CX == BX ise çık
TOP AX 1
ATM [BX] AX         ; AX değerini RAM[BX]'e yaz
ATM CX DX           ; CX'i N'ye geri yükle
TOP BX 1            ; adresi artır
D ETIKET1
ETIKET2: ATM CX DX
ATM BX 1
ETIKET3: CIK CX BX
DE ETIKET4
YAZ [BX]            ; RAM[BX] değerini yazdır
TOP BX 1
ATM CX DX
D ETIKET3
ETIKET4: SON
```
 
## 🖥️ Çalıştırma
 
Programlar, kaynak dosya yolunu **kodun içine sabit (hardcoded)** olarak yazılmış şekilde okur:
 
```java
String filePath = "C:\\Users\\WINUSER\\Desktop\\Program Yapısı ve Anlamı-Dosyalar\\program.txt";
```
 
Kendi ortamınızda çalıştırmak için:
 
1. `Execution.java` içindeki `filePath` değişkenini kendi `program.txt` (veya `Odev3-1.txt` / `Odev3-2.txt`) konumunuza göre güncelleyin.
2. Derleyip çalıştırın:
```bash
   javac Execution.java
   java execution.Execution
```
3. `OKU` komutlarına karşılık gelen girdileri konsoldan girin.
## ⚠️ Bilinen Sınırlamalar
 
- **Dosya yolu sabit kodlanmış:** Komut satırı argümanı veya göreli yol desteği yok; her çalıştırmadan önce kaynak kodda elle değiştirilmesi gerekiyor.
- **Sıfıra bölme kontrolü yok:** `BOL` komutu, kaynak değeri `0` olduğunda Java'nın fırlattığı `ArithmeticException`'ı yakalamaz; program çöker.
- **`Execution.java` ile `Execution3.java` neredeyse birebir aynı:** Aralarındaki tek fark açıklama yorumlarının varlığı/yokluğu ve paket adı — muhtemelen aynı ödevin iki farklı teslim/versiyonu.
- **Girdi doğrulama sınırlı:** `OKU` dışındaki komutlarda geçersiz operand (örn. tanımsız kayıtçı adı) sessizce yok sayılabilir.
## 🎯 Amaç
 
Bu proje, bir **Bilgisayar Mimarisi / Programlama Dilleri Mantığı** dersi kapsamında, düşük seviyeli bir işlemcinin komut çevrimini (fetch-decode-execute), adresleme modlarını ve bayrak mekanizmalarını yüksek seviyeli bir dilde (Java) modelleyerek anlaşılır kılmayı amaçlamaktadır.
