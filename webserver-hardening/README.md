
# Security Hardening Guideline

Konfigurasi berikut adalah contoh security hardening config web server (Nginx) dengan TLS, HTTP security headers, serta resource limiting.

---
## TLS & SSL Hardening
```bash
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256';
ssl_prefer_server_ciphers on;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
```

- Hanya izinkan TLS 1.2 & 1.3  
- Cipher suite aman (AES-GCM & CHACHA20)  
- Server memilih cipher untuk cegah downgrade  
- OCSP Stapling memastikan sertifikat valid  

---
## HTTP Security Headers
```bash
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header Referrer-Policy "strict-origin-when-cross-origin";
add_header Content-Security-Policy "default-src 'self'; script-src 'self'";
```
- HSTS: HTTPS wajib selama 1 tahun  
- X-Frame-Options: Lindungi dari clickjacking  
- X-Content-Type-Options: Cegah MIME sniffing  
- Referrer-Policy: Minimalkan bocoran referer  
- CSP: Hanya izinkan resource dari origin sendiri  

---
## Server Information Disclosure
```bash
server_tokens off;
autoindex off;
```
- Sembunyikan versi server  
- Nonaktifkan directory listing  

---
## Request & Resource Limiting
```bash
client_max_body_size 10M;
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
limit_req zone=one burst=20 nodelay;
```
- Maksimum body 10 MB  
- Batasi request 10/s per IP, burst max 20  

---
## Cookie Security
```bash
proxy_cookie_path / "/; Secure; HttpOnly; SameSite=Strict";
```
- Secure: hanya dikirim via HTTPS  
- HttpOnly: cegah akses JavaScript  
- SameSite=Strict: cegah CSRF  

---

## Rekomendasi Tambahan
- Gunakan sertifikat SSL/TLS valid & update rutin  
- Tambahkan Fail2Ban / WAF untuk filter serangan  
- Lakukan penetration testing & vulnerability scan  
- Integrasi ke CI/CD pipeline untuk monitoring konfigurasi  
