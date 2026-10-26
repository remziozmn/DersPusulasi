# 🧭 DersPusulası — Kayıttan Sınav Hazırlığına

**Ders kayıtlarını yalnızca yazıya dönüştürmeyen; hocanın vurguladığı noktaları yakalayan, önemli anlara geri dönmeyi kolaylaştıran ve isteğe bağlı olarak yerel yapay zekâyla ders notları ve tekrar soruları oluşturan açık kaynak bir Python projesi.**

> Sorun: 90 dakikalık bir ders kaydını tekrar tekrar açmak yerine, “Hoca Observer ve Strategy farkını ne zaman anlattı?”, “Sınav için neye dikkat çekti?” ve “Bu konuyu hatırlıyor muyum?” sorularını cevaplayabilmek.

## ✨ Neler yapar?

- **Perculus vb. platformlardan** erişim yetkin bulunan, süreli/imzalı doğrudan MP4 URL'sini veya yerel ses/video dosyasını işler.
- **Türkçe zaman damgalı transkript** çıkarır (Faster-Whisper, yerel).
- **Sınav/quiz/önem ifadelerini** transparan kurallarla işaretler; ilgili cümlenin önünü/arkasını gösterir.
- **Aranabilir tek HTML dashboard** üretir. İnternet bağlantısı olmadan tarayıcıdan açılır.
- **TXT, SRT, JSON, Markdown** olarak çıktı verir.
- **Opsiyonel Ollama** (yerel LLM) ile başlıklandırılmış ders notları ve cevaplı aktif hatırlama soruları oluşturur; notların yanına ilgili transkript saatlerini ekler.
- Ollama kurmadan da **transkript, zaman çizelgesi, sınav işaretleri ve 5 dakikalık tekrar haritası** hazırlar.

Bu bir **MVP / portföy projesidir**. Kurallar sınav sorularını kesin olarak tahmin etmez; LLM, kaynak ID'sini doğrulasa da içerik anlamında yanılabilir. Kullanıcı önemli noktaları video ile kontrol etmelidir.

## 🏗️ Mimari

```text
Yetkili MP4 URL'si / yerel video
        |
        v
Geçici indirici (requests, HTTP(S))
        |
        v
Faster-Whisper (Türkçe STT)
        |
        v
Zaman damgalı segmentler (JSON)
        |                    \
        v                     v
Kural tabanlı sınav analizi    Ollama (isteğe bağlı, yerel)
        |                     | 
        v                     v
İşaretler + bağlam            Notlar + tekrar soruları
        \                     /
         \                   /
          v                 v
          Aranabilir HTML dashboard
          + TXT / SRT / JSON / Markdown
```

**Teknolojiler:** Python 3.10+, Requests, Faster-Whisper, PyAV (bağımlılık olarak), standart kütüphane HTML üretimi, opsiyonel Ollama HTTP API, unittest.

## 🚀 Windows üzerinde kurulum

Python 3.10+ kurulu olmalı. Terminali **proje kök klasöründe** aç:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -e .
```

PowerShell betik politikası etkinleştirmeye izin vermiyorsa aktivasyon gerekmeden `.venv\Scripts\python.exe -m pip install -e .` komutunu kullanabilirsin.

**Kurulumu videoya ihtiyaç olmadan test et:**

```powershell
python -m derspusulasi demo
```

Terminal çıktısındaki `outputs/.../dashboard.html` dosyasını çift tıklayıp aç. Bu komut **tamamen uydurma bir örnek ders** kullanır; gerçek hocanın içeriği değildir.

## 🎞️ Perculus 7 MP4 bağlantısıyla çalışma

Yalnızca erişim iznin bulunan ders kayıtları için geçerlidir. Tarayıcının **F12 → Network** sekmesinde tespit edilen doğrudan `.mp4?...` URL'sini **adres çubuğunda / kurumun erişim koşullarına göre** doğrula. İmzalı URL'ler süreli olabilir.

URL'yi `--url` ile komut satırına koymak yerine **etkileşimli/gizli giriş** kullan; böylece bağlantı shell geçmişine yazılmaz:

```powershell
python -m derspusulasi run --title "Design Patterns Hafta 1" --course "Design Patterns"
```

`Yetkili video URL'sini yapıştır` sorulduğunda bağlantıyı gir ve Enter'a bas. **Güvenlik nedeniyle yapıştırdığın metin terminalde görünmez.** Bağlantı geçici alınır ve işlem sonunda video silinir.

Yerel MP4/MP3 dosyan varsa:

```powershell
python -m derspusulasi run --file "C:\\Dersler\\hafta1.mp4" --title "Hafta 1"
```

Daha iyi fakat daha ağır bir model denemek için `--model medium` (varsayılan `small`). GPU kullanımı yalnızca uyumlu CUDA/CuDNN kurulumunda `--device cuda` ile mümkündür.

### ⚠️ URL sorunları

- `HTTP 403 / AccessDenied`: Bağlantı süresi dolmuş veya erişim yetkisi yok; platformdan yeni yetkili bağlantı edin.
- `HTTP 404`: Medya URL'si artık mevcut olmayabilir.
- `model not found` vb.: İlk kullanımda model internetten indirilir; disk ve ağ erişimini denetle.
- Hoca `screen share` kullanıyorsa **yalnızca ses transkripsiyonu** yapılır; slayt üzerindeki yazılar okunmaz.
- MP4 dosyası şifreli/DRM korumalıysa bu yazılım korumayı aşmaz.

## 🧠 Yapay zekâ destekli not ve tekrar soruları

Bu aşama **isteğe bağlıdır**. [Ollama](https://ollama.com/download) yükle, çalıştır ve model indir:

```powershell
ollama pull qwen2.5:3b
```

Ardından:

```powershell
python -m derspusulasi run --file "C:\\Dersler\\hafta1.mp4" --title "Hafta 1" --ollama-model qwen2.5:3b
```

**Aynı uzun videoyu tekrar transkribe etmeden** önceki `segments.json` dosyasını kullan:

```powershell
python -m derspusulasi analyze --json "outputs\hafta1_YYYYMMDD_HHMMSS\segments.json" --title "Hafta 1" --ollama-model qwen2.5:3b
```

`YYYYMMDD_HHMMSS` bir örnektir; gerçek oluşan klasör adını kullan. Ollama **bilgisayarındaki** `http://localhost:11434/api/generate` API'si üzerinden çalışır. Uzun dersler birkaç parçaya bölünür. Yerel LLM'nin RAM ve işlem süresi ihtiyacı vardır; zayıf bir bilgisayarda yalnızca temel analiz modu kullanılabilir.

**Kaynağa dayalı üretim:** AI'ya konuşma parçaları zaman damgalı ID'lerle gönderilir. Yanıttaki kaynak ID'leri program tarafından gerçekten var olan cümlelerle doğrulanır. Bu, *alıntılanan cümlenin AI yorumunu mutlaka doğruladığı anlamına gelmez.* Yanlış yorum ve uydurma yine mümkündür.

## 📁 Oluşan dosyalar

```text
outputs/
└── design-patterns-hafta-1_20261026_150000/
    ├── dashboard.html          # Tarayıcıda açılan çalışma paneli
    ├── transkript.txt          # Tüm Türkçe konuşmalar + zaman damgaları
    ├── segments.json           # Yeniden analiz edilebilir yapılandırılmış veri
    ├── altyazi.srt             # Altyazı dosyası
    ├── sinav_isaretleri.md     # Öğretmenin vurgu/sınav ifadeleri
    ├── tekrar_haritasi.md      # 5 dakikalık kısa alıntı haritası
    ├── yapay_zeka_notlari.md   # Sadece Ollama açıksa
    └── calisma_verisi.json     # Sadece Ollama açıksa
```

`.gitignore` bu çıktıları ve ses/video dosyalarını **GitHub'a yüklememek** üzere ayarlar.

## 🧪 Test

```powershell
python -m unittest discover -s tests -v
```

Testler gerçek video veya Whisper modeli indirmez; örnek veriler ve sahte HTTP yanıtları kullanılır.

## 🔏 Veri güvenliği ve etik

- Sadece **yasal olarak erişebildiğin ve işleme iznin olan** kayıtlar için kullan.
- Erişim belirteci içeren süreli linkleri **README, issue, commit, terminal geçmişi ve ekran görüntüsüne koyma**. Bu proje URL'yi kaydetmez; HTTP hata mesajlarındaki URL'yi de göstermez.
- Öğretim elemanının sesi ve ders içeriği kişisel veri/telif kapsamına girebilir; gerçek ders transkriptlerini ve medyayı açık depoya yükleme.
- Bu araç şifreleme, oturum açma, DRM veya yetki denetimini aşmayı amaçlamaz.
- İmzalı video bağlantısı girdikten sonra medya geçici klasöre indirilir. **Transkript/Ollama kullanımı yereldir**; medya indirilmesi için kaynak sunucuya, modelin ilk kurulumu için model sunucusuna ağ erişimi gerekir.

## 🗺️ Yol haritası

- [x] Türkçe zamanlı transkript ve altyazı
- [x] Öğretmen vurgu/sınav işareti tespiti
- [x] Aranabilir tek dosyalık HTML dashboard
- [x] İsteğe bağlı yerel LLM ile kaynaklı not ve quiz
- [x] Yeniden transkripsiyon gerektirmeyen analiz
- [ ] Bağlama duyarlı sınav ifadesi sınıflandırması (false positive azaltma)
- [ ] Slayt OCR ve ses/slayt eşleştirme
- [ ] Çok derslik klasör, ders bazlı arama ve konu indeksi
- [ ] Her kart için aralıklı tekrar planı (spaced repetition)
- [ ] Kullanıcı dostu masaüstü / Streamlit arayüzü

## 👨‍💻 GitHub'a yükleme

Önce `git` kuruluysa proje klasöründe:

```powershell
git init
git add .
git commit -m "feat: initial DersPusulasi MVP"
git branch -M main
git remote add origin https://github.com/KULLANICI_ADI/derspusulasi.git
git push -u origin main
```

Önce GitHub'da boş, **README eklenmemiş** bir `derspusulasi` deposu aç. `KULLANICI_ADI` kısmını değiştir. Video ve `outputs` klasörleri GitHub'a yüklenmez.

## Kaynaklar

- [Faster-Whisper projesi](https://github.com/SYSTRAN/faster-whisper)
- [Ollama yerel API](https://github.com/ollama/ollama/blob/main/docs/api.md)

## Lisans

MIT. Bu lisans yalnızca bu depodaki **kaynak kodun** kullanımına ilişkindir; ders kayıtları, video, konuşma metinleri ve diğer üçüncü taraf içeriklerine hak sağlamaz.