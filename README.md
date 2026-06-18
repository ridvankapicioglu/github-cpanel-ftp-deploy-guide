# GitHub Actions ile cPanel FTP Deploy Kurulumu

Bu rehber, SSH veya Terminal erişimi olmayan cPanel hostinglerde GitHub Actions kullanarak otomatik FTP Deploy kurulumu yapmak için hazırlanmıştır.

Bu yöntem sayesinde her `git push` işleminden sonra dosyalar otomatik olarak cPanel üzerindeki hedef klasöre aktarılır.

---

# Sistem Mimarisi

```text
Bilgisayar
    ↓ git push
GitHub Repository
    ↓ GitHub Actions
FTP Deploy
    ↓
cPanel Hosting
```

Bu yapı sayesinde:

* Kodlar GitHub üzerinde versiyonlanır.
* Yapılan değişiklikler kayıt altına alınır.
* Manuel ZIP yükleme ihtiyacı ortadan kalkar.
* Her deploy işlemi GitHub Actions tarafından otomatik gerçekleştirilir.

---

# Gereksinimler

* GitHub hesabı
* cPanel hosting
* FTP erişimi
* Git kurulu bilgisayar
* GitHub Repository

---

# 1. FTP Hesabı Oluşturma

cPanel üzerinden:

```text
FTP Accounts
```

menüsüne girin.

Projenin bulunduğu klasöre erişebilen bir FTP hesabı oluşturun.

Örnek:

```text
Kullanıcı Adı: ridvan
Şifre: ********
Dizin: /home2/kullaniciadi/deneme.ridvan.com
```

FTP hesabı doğrudan hedef klasöre erişebiliyorsa ileride:

```yaml
server-dir: /
```

kullanılabilir.

---

# 2. GitHub Secrets Oluşturma

Repository içerisinde:

```text
Settings
→ Secrets and variables
→ Actions
→ New repository secret
```

Aşağıdaki secretları oluşturun:

```text
FTP_SERVER
FTP_USERNAME
FTP_PASSWORD
```

Örnek:

```text
FTP_SERVER   = ftp.domain.com
FTP_USERNAME = ftp_kullanici
FTP_PASSWORD = ftp_sifre
```

Bu bilgiler GitHub tarafından şifreli şekilde saklanır.

---

# 3. Workflow Klasörünü Oluşturma

Proje kök dizininde aşağıdaki klasör yapısını oluşturun:

```text
.github/
└── workflows/
```

---

# 4. Deploy Workflow Oluşturma

Dosya:

```text
.github/workflows/deploy.yml
```

İçeriği:

```yaml
name: Deploy to cPanel via FTP

on:
  push:
    branches:
      - main

jobs:
  ftp-deploy:
    name: Deploy via FTP
    runs-on: ubuntu-latest

    steps:
      - name: Get latest code
        uses: actions/checkout@v4

      - name: Deploy files
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          local-dir: ./
          server-dir: /
          protocol: ftps
          port: 21
          dry-run: true

          exclude: |
            **/.git*
            **/.git*/**
            **/.github/**
            **/.vscode/**
            **/.idea/**
            **/.claude/**
            **/.env
            **/.env.*
            **/node_modules/**
            **/vendor/**
            **/*.zip
            **/*.rar
            **/*.7z
            **/*.apk
            **/*.log
```

---

# 5. İlk Test (Dry Run)

İlk aşamada:

```yaml
dry-run: true
```

aktif bırakılır.

Bu modda:

* Sunucuya dosya gönderilmez.
* Yapılacak işlemler loglarda görüntülenir.
* Workflow güvenli şekilde test edilir.

Komutlar:

```bash
git add .
git commit -m "Add FTP deploy workflow"
git push
```

GitHub üzerinde:

```text
Actions
```

sekmesinden loglar incelenebilir.

---

# 6. Deploy'u Aktifleştirme

Test başarılıysa:

```yaml
dry-run: true
```

satırı kaldırılır.

Sonrasında:

```bash
git add .
git commit -m "Enable FTP deploy"
git push
```

çalıştırılır.

Artık her:

```bash
git push
```

işleminde GitHub Actions otomatik olarak deploy işlemini başlatacaktır.

---

# 7. Laravel Projeleri İçin Önerilen .gitignore

```gitignore
/vendor
/node_modules

.env
.env.*
!.env.example

/storage/logs/*
/storage/framework/cache/*
/storage/framework/sessions/*
/storage/framework/testing/*
/storage/framework/views/*

/public/uploads/
/public/apk/

*.apk
*.zip
*.rar
*.7z

.idea/
.vscode/

.claude/

.DS_Store
Thumbs.db

.phpunit.result.cache
```

---

# 8. Exclude Listesi

Deploy sırasında aşağıdaki dosyaların gönderilmemesi önerilir:

```yaml
exclude: |
  **/.git*
  **/.git*/**
  **/.github/**
  **/.vscode/**
  **/.idea/**
  **/.claude/**

  **/.env
  **/.env.*
  !**/.env.example

  **/vendor/**
  **/node_modules/**

  **/public/uploads/**
  **/public/apk/**

  **/*.apk
  **/*.zip
  **/*.rar
  **/*.7z

  **/storage/logs/**
  **/storage/framework/cache/**
  **/storage/framework/sessions/**
  **/storage/framework/testing/**
  **/storage/framework/views/**

  **/__MACOSX/**
  **/.DS_Store

  **/npm-debug.log
  **/yarn-error.log
```

---

# 9. Günlük Kullanım

Çalışmaya başlamadan önce:

```bash
git pull
```

Değişiklik yaptıktan sonra:

```bash
git add .
git commit -m "Yapılan değişiklik açıklaması"
git push
```

GitHub Actions otomatik olarak deploy işlemini gerçekleştirecektir.

---

# 10. Önemli Notlar

## .env Dosyası

`.env` dosyası GitHub'a gönderilmemelidir.

```gitignore
.env
.env.*
```

şeklinde korunmalıdır.

## Upload Klasörleri

Kullanıcı tarafından yüklenen dosyalar:

```text
public/uploads
```

klasöründe tutuluyorsa GitHub'a gönderilmemelidir.

## APK Dosyaları

Android uygulama çıktıları:

```text
*.apk
```

GitHub repository içerisine eklenmemelidir.

## dangerous-clean-slate

Bu ayar kullanılmamalıdır.

Aksi halde GitHub'da bulunmayan dosyalar sunucudan silinebilir.

---

# Sonuç

Bu yapı sayesinde:

* GitHub merkezi kaynak olur.
* FTP ile manuel dosya yükleme ihtiyacı azalır.
* Her değişiklik kayıt altına alınır.
* Deploy işlemi otomatikleşir.
* cPanel paylaşımlı hostinglerde profesyonel bir yayınlama süreci elde edilir.
