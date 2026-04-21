# 🎨 Guida Open-WebUI - Configurazione Completa

## 📋 Come Funziona l'Integrazione

Open-WebUI è configurato per usare **DUE sorgenti di modelli**:

```
┌─────────────────────────────────────────────────────────────┐
│                       Open-WebUI                            │
│                    (localhost:3055)                         │
└────────────┬──────────────────────────────┬─────────────────┘
             │                              │
             │                              │
    ┌────────▼────────┐            ┌────────▼────────────┐
    │     Ollama      │            │   LiteLLM Proxy     │
    │  (localhost)    │            │  (localhost:4000)   │
    │                 │            │                     │
    │  • llama3.2     │            │  • Claude (paid)    │
    │  • codellama    │            │  • GPT (paid)       │
    │  • gemma3       │            │  • Ollama (local)   │
    │  • mistral      │            │  • Azure, etc.      │
    └─────────────────┘            └─────────────────────┘
         LOCALE                      MULTI-PROVIDER
         GRATIS                      (alcune a pagamento)
```

---

## 🔧 Configurazione in Open-WebUI

### Passo 1: Accedi a Open-WebUI

1. Apri: **http://localhost:3055**
2. **Primo accesso**: Crea un account (diventerai automaticamente admin)
3. Dopo il primo utente, le registrazioni sono **disabilitate** per sicurezza

### Passo 2: Configura Connessioni

#### A) Modelli Ollama Locali (GIÀ CONFIGURATO ✅)

Open-WebUI vede automaticamente i modelli Ollama:
- ✅ llama3.2:3b
- ✅ llama3.2:latest
- ✅ codellama:latest
- ✅ gemma3:latest
- ✅ embeddinggemma:latest

**Questi funzionano subito, nessuna configurazione richiesta!**

#### B) Modelli via LiteLLM (Claude, GPT, etc.)

Per usare modelli esterni (Claude, GPT, ecc.) tramite LiteLLM:

1. **In Open-WebUI**, vai a:
   - Click sull'icona del profilo (in alto a destra)
   - **Settings** → **Connections** (o **Admin Panel** → **Settings**)

2. **Aggiungi OpenAI API** (LiteLLM è compatibile):
   ```
   OpenAI API Base URL: http://litellm:4000/v1
   OpenAI API Key: ${LITELLM_MASTER_KEY}
   ```

3. **Salva** e ricarica la pagina

---

## 🔑 Come Abilitare Claude/GPT in LiteLLM

I modelli Claude che vedi nel database di LiteLLM sono **disponibili**, ma **non funzionano senza chiavi API**.

### Per usare Claude (Anthropic):

1. **Ottieni una API key**:
   - Vai su: https://console.anthropic.com/
   - Crea un account
   - Genera una API key
   - **Costa**: ~$15 per 1M token input (Claude Sonnet)

2. **Aggiungi la chiave nel file `.env`**:
   ```bash
   nano /opt/litellm/.env
   ```

   Decommenta e aggiungi:
   ```bash
   ANTHROPIC_API_KEY="sk-ant-tua-chiave-qui"
   ```

3. **Riavvia LiteLLM**:
   ```bash
   docker compose restart litellm
   ```

4. **Ora Claude funzionerà** sia via API LiteLLM che in Open-WebUI!

### Per usare OpenAI (GPT):

1. **Ottieni una API key**:
   - Vai su: https://platform.openai.com/
   - Crea un account e aggiungi credito
   - Genera una API key
   - **Costa**: ~$15 per 1M token input (GPT-4o)

2. **Aggiungi nel `.env`**:
   ```bash
   OPENAI_API_KEY="sk-tua-chiave-openai-qui"
   ```

3. **Riavvia**:
   ```bash
   docker compose restart litellm
   ```

---

## 🎯 Come Vedere i Modelli in Open-WebUI

### Opzione 1: Via Interfaccia Web

1. Apri Open-WebUI: http://localhost:3055
2. Nella chat, clicca sul **selettore modelli** (in alto)
3. Dovresti vedere:
   - **Sezione Ollama**: modelli locali
   - **Sezione OpenAI** (se configurato): modelli via LiteLLM

### Opzione 2: Configura Manualmente le Connessioni

Se i modelli LiteLLM non appaiono automaticamente:

1. **Settings** → **Admin Panel** → **Connections**

2. **Aggiungi OpenAI Connection**:
   - URL: `http://litellm:4000/v1`
   - API Key: `${LITELLM_MASTER_KEY}`
   - ✅ Enable

3. **Verifica disponibilità modelli**:
   - Settings → Admin Panel → **Models**
   - Dovresti vedere tutti i modelli configurati in LiteLLM

---

## 💡 Perché Vedi Modelli Claude ma Non Funzionano?

LiteLLM ha i modelli Claude **pre-configurati nel database**, ma sono **inattivi** perché:

1. ❌ **Non hai una chiave API di Anthropic** nel file `.env`
2. ❌ Senza chiave, LiteLLM non può contattare i server di Anthropic
3. ✅ Soluzione: Aggiungi la chiave API (vedi sopra) oppure usa solo modelli Ollama locali

---

## 🆓 Usa Solo Modelli Locali (Gratis)

Se **non vuoi pagare** per Claude/GPT, usa solo i modelli Ollama locali:

### Modelli Consigliati per RTX 3080 (10GB):

```bash
# Già installati:
docker exec ollama ollama list

# Scarica altri modelli gratis:
docker exec ollama ollama pull mistral:7b           # Ottimo generale
docker exec ollama ollama pull qwen2.5:7b          # Eccellente per coding
docker exec ollama ollama pull deepseek-coder:6.7b # Specializzato coding
docker exec ollama ollama pull llava:latest        # Per analizzare immagini
docker exec ollama ollama pull phi3:medium         # Leggero e veloce

# Modelli più grandi (usano quasi tutta la VRAM):
docker exec ollama ollama pull llama3.1:70b-q4     # Quantizzato 4-bit
```

### Poi aggiungili a LiteLLM:

```bash
# Usa lo script automatico
./add_ollama_models.sh

# Oppure aggiungi manualmente ogni nuovo modello:
curl -X POST http://localhost:4000/model/new \
  -H "Authorization: Bearer ${LITELLM_MASTER_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "model_name": "mistral",
    "litellm_params": {
      "model": "ollama_chat/mistral:7b",
      "api_base": "http://ollama:11434"
    }
  }'
```

---

## 🔍 Verifica Configurazione Open-WebUI

### 1. Verifica Connessione a Ollama

In Open-WebUI:
- Crea una nuova chat
- Seleziona modello: `llama3.2:3b` o `llama3.2:latest`
- Scrivi: "Ciao!"
- ✅ **Dovrebbe rispondere immediatamente** (modello locale)

### 2. Verifica Connessione a LiteLLM

Prima assicurati che LiteLLM sia configurato in Open-WebUI:

**Settings → Admin Panel → Connections → OpenAI API**:
```
Base URL: http://litellm:4000/v1
API Key: ${LITELLM_MASTER_KEY}
```

Poi:
- Seleziona modello: `llama3.2-3b` (tramite LiteLLM)
- Scrivi: "Test"
- ✅ Dovrebbe funzionare (stesso modello, ma tramite proxy)

### 3. Verifica Modelli Disponibili

In Open-WebUI, nel selettore modelli dovresti vedere:

**Ollama (modelli locali)**:
- llama3.2:3b
- llama3.2:latest
- codellama:latest
- gemma3:latest
- embeddinggemma:latest

**OpenAI/LiteLLM** (se configurato):
- llama3.2-3b (tramite LiteLLM)
- llama3.2 (tramite LiteLLM)
- codellama (tramite LiteLLM)
- gemma3 (tramite LiteLLM)
- claude-sonnet-4 (⚠️ richiede ANTHROPIC_API_KEY)
- claude-opus-4 (⚠️ richiede ANTHROPIC_API_KEY)
- gpt-4 (⚠️ richiede OPENAI_API_KEY)

---

## 🎨 Features Avanzate Open-WebUI

### 1. RAG (Retrieval Augmented Generation)

Carica documenti per farli analizzare dall'AI:

1. Click su **"+"** → **Upload Document**
2. Carica PDF, DOCX, TXT, MD
3. Seleziona modello embedding: `embeddinggemma:latest`
4. Fai domande sui documenti caricati

### 2. Web Search

Abilita ricerca web (se configurata):
- Settings → Features → Web Search
- Permette all'AI di cercare informazioni online

### 3. Image Generation

Se configuri un modello di generazione immagini:
- Ollama: `docker exec ollama ollama pull stable-diffusion`
- Oppure usa DALL-E tramite OpenAI (a pagamento)

### 4. Voice Input

Open-WebUI supporta input vocale:
- Settings → Audio → Enable Speech-to-Text
- Richiede configurazione Whisper o servizi cloud

---

## 🆘 Troubleshooting

### "No models available" in Open-WebUI

**Soluzione**:
```bash
# Verifica che Ollama funzioni
docker exec ollama ollama list

# Riavvia Open-WebUI
docker compose restart ui

# Controlla logs
docker compose logs ui
```

### Modelli LiteLLM non visibili

**Soluzione**:
1. Verifica che LiteLLM funzioni:
   ```bash
   curl http://localhost:4000/model/info \
     -H "Authorization: Bearer ${LITELLM_MASTER_KEY}"
   ```

2. Configura connessione OpenAI in Open-WebUI:
   - Settings → Connections
   - URL: `http://litellm:4000/v1`
   - Key: `${LITELLM_MASTER_KEY}`

### Claude/GPT danno errore "Invalid API Key"

**Causa**: Non hai configurato le chiavi API nel file `.env`

**Soluzione**:
1. Ottieni chiavi API da Anthropic/OpenAI
2. Aggiungi nel `.env`
3. Riavvia: `docker compose restart litellm`

---

## 📊 Riepilogo: Cosa Usare

| Scenario | Soluzione | Costo |
|----------|-----------|-------|
| **Chat generale gratis** | Ollama: llama3.2, mistral | 🆓 Gratis |
| **Coding gratis** | Ollama: codellama, qwen2.5 | 🆓 Gratis |
| **Massima qualità (paid)** | Claude Sonnet 4 via LiteLLM | 💰 ~$15/1M token |
| **Analisi immagini gratis** | Ollama: llava | 🆓 Gratis |
| **Embeddings per RAG** | Ollama: embeddinggemma | 🆓 Gratis |
| **Multi-provider + fallback** | LiteLLM Proxy | 🆓 + 💰 (dipende) |

---

## ✅ Riepilogo Setup Ideale

**Configurazione consigliata per iniziare (GRATIS)**:

1. ✅ **Usa solo modelli Ollama locali**
2. ✅ Scarica modelli utili: `mistral:7b`, `qwen2.5:7b`
3. ✅ Configura RAG con `embeddinggemma`
4. ✅ Testa tutto tramite Open-WebUI

**Quando sei pronto per modelli cloud (A PAGAMENTO)**:

1. 💰 Ottieni API key di Anthropic o OpenAI
2. 💰 Aggiungi nel `.env`
3. 💰 Riavvia LiteLLM
4. 💰 Usa Claude/GPT per task complessi

---

**La configurazione attuale ti permette di usare modelli AI potenti completamente GRATIS sulla tua GPU! 🚀**

Per domande: consulta [SETUP_GUIDA.md](./SETUP_GUIDA.md) e [ESEMPI_USO.md](./ESEMPI_USO.md)
