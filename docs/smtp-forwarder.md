# SMTP Forwarder - Proiect Go

## Descriere Generală

SMTP Forwarder este un server proxy/relay pentru email-uri, implementat în Go, care acționează ca intermediar între clienții SMTP și un server SMTP extern. Aplicația primește conexiuni SMTP de la clienți autentificați și retransmite mesajele către un server SMTP de destinație configurat.

## Milestone 1: Funcționalitate de Bază

### Obiective

- Implementarea unui server SMTP funcțional care ascultă pe un port configurabil
- Autentificare simplă pentru clienții care se conectează
- Forwarding complet al mesajelor către serverul SMTP extern
- Suport pentru conexiuni TLS/STARTTLS
- Configurare prin fișier `.env`

### Arhitectura Sistemului

```
┌──────────────┐         ┌─────────────────┐         ┌──────────────────┐
│  Client SMTP │  ──-->  │  SMTP Forwarder │  ──-->  │ Server SMTP      │
│  (Outlook,   │         │   (Go App)      │         │ Extern           │
│  Thunderbird,│         │                 │         │ (smtp2go.com)    │
│  PHPMailer)  │         │  Port: 2587     │         │                  │
└──────────────┘         └─────────────────┘         └──────────────────┘
       │                          │                           │
       │    AUTH + SEND          │      RELAY WITH           │
       └─────────────────────────>│      NEW AUTH            │
                                  └──────────────────────────>│
```

### Structura Proiectului

```
smtp-forwarder/
├── cmd/
│   └── server/
│       └── main.go           # Entry point
├── internal/
│   ├── config/
│   │   └── config.go         # Configurare din .env
│   ├── smtp/
│   │   ├── server.go         # Server SMTP principal
│   │   ├── session.go        # Gestionare sesiune SMTP
│   │   └── forwarder.go      # Logica de forwarding
│   └── auth/
│       └── auth.go           # Autentificare
├── pkg/
│   └── logger/
│       └── logger.go         # Logging
├── .env.example              # Exemplu configurare
├── .env                      # Configurare locală (ignorat în git)
├── .gitignore
├── go.mod
├── go.sum
├── Makefile
└── README.md
```

## Implementare

### 1. Configurarea Mediului (config/config.go)

```go
package config

import (
    "github.com/joho/godotenv"
    "os"
    "strconv"
)

type Config struct {
    // Server ingress (incoming)
    IngestionPort     int
    IngestionSecure   string
    IngestionUsername string
    IngestionPassword string
    
    // Server extern (outgoing)
    ExternalHost     string
    ExternalPort     int
    ExternalSecure   string
    ExternalUsername string
    ExternalPassword string
}

func Load() (*Config, error) {
    if err := godotenv.Load(); err != nil {
        return nil, err
    }
    
    return &Config{
        IngestionPort:     getEnvAsInt("INGESTION_SMTP_PORT", 2587),
        IngestionSecure:   getEnv("INGESTION_SMTP_SECURE", "tls"),
        IngestionUsername: getEnv("INGESTION_SMTP_USERNAME", ""),
        IngestionPassword: getEnv("INGESTION_SMTP_PASSWORD", ""),
        
        ExternalHost:     getEnv("EXTERNAL_SMTP_HOST", ""),
        ExternalPort:     getEnvAsInt("EXTERNAL_SMTP_PORT", 587),
        ExternalSecure:   getEnv("EXTERNAL_SMTP_SECURE", "tls"),
        ExternalUsername: getEnv("EXTERNAL_SMTP_USERNAME", ""),
        ExternalPassword: getEnv("EXTERNAL_SMTP_PASSWORD", ""),
    }, nil
}
```

### 2. Server SMTP Principal

Serverul utilizează biblioteca `github.com/emersion/go-smtp` pentru implementarea protocolului SMTP:

```go
package smtp

import (
    "github.com/emersion/go-smtp"
    "crypto/tls"
)

type Server struct {
    config    *config.Config
    smtpServer *smtp.Server
}

func NewServer(cfg *config.Config) *Server {
    be := &Backend{config: cfg}
    
    s := smtp.NewServer(be)
    s.Addr = fmt.Sprintf(":%d", cfg.IngestionPort)
    s.Domain = "localhost"
    s.AllowInsecureAuth = false
    
    // Configurare TLS dacă e necesar
    if cfg.IngestionSecure == "tls" {
        s.TLSConfig = &tls.Config{
            // Configurare certificat TLS
        }
    }
    
    return &Server{
        config: cfg,
        smtpServer: s,
    }
}

func (s *Server) Start() error {
    log.Printf("Starting SMTP server on port %d", s.config.IngestionPort)
    return s.smtpServer.ListenAndServe()
}
```

### 3. Backend și Sesiuni

```go
type Backend struct {
    config *config.Config
}

func (b *Backend) NewSession(c *smtp.Conn) (smtp.Session, error) {
    return &Session{
        config: b.config,
        conn: c,
    }, nil
}

type Session struct {
    config *config.Config
    conn   *smtp.Conn
    from   string
    to     []string
}

func (s *Session) AuthPlain(username, password string) error {
    // Verificare credențiale
    if username != s.config.IngestionUsername || 
       password != s.config.IngestionPassword {
        return errors.New("Invalid credentials")
    }
    return nil
}

func (s *Session) Mail(from string, opts *smtp.MailOptions) error {
    s.from = from
    return nil
}

func (s *Session) Rcpt(to string, opts *smtp.RcptOptions) error {
    s.to = append(s.to, to)
    return nil
}

func (s *Session) Data(r io.Reader) error {
    // Citire mesaj
    msg, err := io.ReadAll(r)
    if err != nil {
        return err
    }
    
    // Forward către serverul extern
    return s.forwardMessage(msg)
}
```

### 4. Forwarding Logic

```go
func (s *Session) forwardMessage(msg []byte) error {
    auth := smtp.PlainAuth(
        "",
        s.config.ExternalUsername,
        s.config.ExternalPassword,
        s.config.ExternalHost,
    )
    
    addr := fmt.Sprintf("%s:%d", 
        s.config.ExternalHost, 
        s.config.ExternalPort)
    
    // Trimite email către serverul extern
    return smtp.SendMail(
        addr,
        auth,
        s.from,
        s.to,
        msg,
    )
}
```

## Dependențe

În `go.mod`:

```go
module github.com/yourusername/smtp-forwarder

go 1.21

require (
    github.com/emersion/go-smtp v0.20.1
    github.com/joho/godotenv v1.5.1
    github.com/sirupsen/logrus v1.9.3
)
```

## Instalare și Rulare

### 1. Clonare și Setup

```bash
git clone https://github.com/yourusername/smtp-forwarder
cd smtp-forwarder
cp .env.example .env
# Editează .env cu configurările tale
```

### 2. Instalare Dependențe

```bash
go mod download
```

### 3. Build și Rulare

```bash
# Development
go run cmd/server/main.go

# Production build
go build -o smtp-forwarder cmd/server/main.go
./smtp-forwarder
```

### 4. Docker (Opțional)

```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go mod download
RUN go build -o smtp-forwarder cmd/server/main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/smtp-forwarder .
COPY --from=builder /app/.env .
CMD ["./smtp-forwarder"]
```

## Testare

### Test Manual cu Telnet

```bash
telnet localhost 2587
EHLO localhost
AUTH PLAIN <base64_encoded_credentials>
MAIL FROM:<test@example.com>
RCPT TO:<recipient@example.com>
DATA
Subject: Test
From: test@example.com
To: recipient@example.com

Test message
.
QUIT
```

### Test cu Client PHP

```php
<?php
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\SMTP;

$mail = new PHPMailer(true);

$mail->isSMTP();
$mail->Host       = 'localhost';
$mail->SMTPAuth   = true;
$mail->Username   = '<incoming user>';
$mail->Password   = '<incoming password>';
$mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;
$mail->Port       = 2587;

$mail->setFrom('from@example.com', 'Test Sender');
$mail->addAddress('to@example.com', 'Test Recipient');
$mail->Subject = 'Test prin SMTP Forwarder';
$mail->Body    = 'Acesta este un test';

$mail->send();
```

## Securitate

### Considerații Importante

1. **Autentificare Obligatorie**: Serverul nu acceptă conexiuni neautentificate
2. **TLS/STARTTLS**: Suport pentru conexiuni criptate
3. **Rate Limiting**: De implementat în milestone-uri viitoare
4. **Validare Input**: Verificarea adreselor email și a conținutului
5. **Logging**: Toate conexiunile și tentativele sunt loggate

### Certificate TLS

Pentru producție, generează certificate valide:

```bash
# Pentru development, poți genera certificate self-signed
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

## Monitorizare și Logging

Aplicația loghează:
- Conexiuni acceptate/refuzate
- Tentative de autentificare
- Email-uri procesate (from, to, timestamp)
- Erori de forwarding
- Metrici de performanță

Exemplu format log:
```
2024-01-15 10:30:45 [INFO] Server started on port 2587
2024-01-15 10:31:02 [INFO] New connection from 192.168.1.100
2024-01-15 10:31:03 [INFO] Auth successful for user: incoming_user
2024-01-15 10:31:05 [INFO] Email forwarded: from=test@example.com to=dest@example.com size=2048
```

## Troubleshooting

### Probleme Comune

1. **Port în uz**: Verifică că portul nu este ocupat
   ```bash
   lsof -i :2587
   ```

2. **Erori de autentificare**: Verifică credențialele în `.env`

3. **Conexiune refuzată la serverul extern**: 
   - Verifică firewall
   - Verifică credențialele externe
   - Testează conexiunea direct:
   ```bash
   telnet mail.smtp2go.com 587
   ```

4. **Certificate TLS**: Pentru development, setează `INGESTION_SMTP_SECURE=none`

## Roadmap

### Milestone 1 (Current) ✅
- [x] Server SMTP de bază
- [x] Autentificare simplă
- [x] Forwarding către server extern
- [x] Configurare prin .env
- [x] Logging de bază

### Milestone 2 (Planned)
- [ ] Rate limiting per IP/user
- [ ] Queue pentru retry în caz de eșec
- [ ] Persistență mesaje (backup local)
- [ ] Web dashboard pentru monitorizare
- [ ] Metrici Prometheus

### Milestone 3 (Future)
- [ ] Multiple servere externe (load balancing)
- [ ] Filtrare spam
- [ ] Modificare headers
- [ ] Webhook-uri pentru evenimente
- [ ] API REST pentru management

## Licență

MIT License

## Contribuții

Pull requests sunt binevenite. Pentru schimbări majore, deschide mai întâi un issue pentru discuții.

## Contact

Pentru întrebări sau suport, contactează echipa de dezvoltare.