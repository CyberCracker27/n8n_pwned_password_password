# 🔐 NGP-013 — AI-Assisted Pwned Password Checker

**A privacy-aware password security automation workflow built with n8n, Telegram, Google Gemini AI, and the Have I Been Pwned API.**




\


## 📌 Overview

NGP-013 is an AI-assisted password security checker that automates password exposure checking through a Telegram bot. It combines cryptographic hashing, the Have I Been Pwned (HIBP) Pwned Passwords API, JavaScript-based result comparison, and Google Gemini AI to generate understandable security recommendations.

The project was developed as a practical learning exercise during company training with Infoz Anit. The goal was to understand how automation nodes, external APIs, cryptography, and AI services can work together in a cybersecurity workflow.

## 🎯 Objectives

- Automate password exposure checking using n8n.
- Integrate Telegram as the user interaction interface.
- Use SHA-1 hashing for compatibility with the HIBP Pwned Passwords API.
- Apply the API's k-anonymity approach to minimise information disclosure.
- Generate concise security recommendations using Google Gemini AI.
- Explore secure data handling and privacy-aware automation.

## ⚙️ Workflow Architecture

The workflow follows this sequence:

```text
Telegram User
     |
     v
Telegram Trigger
     |
     v
Edit Fields
     |
     v
Crypto — SHA-1 Hash
     |
     v
Code in JavaScript
(Split Hash: Prefix + Suffix)
     |
     v
HTTP Request
(HIBP Range API)
     |
     v
Code in JavaScript1
(Compare Hash Suffix)
     |
     v
Edit Fields1
(Minimise AI Input)
     |
     v
AI Agent <---- Google Gemini Chat Model
     |
     v
Telegram — Send Security Guidance
```

### 🔄 How It Works

**1. Telegram Trigger**

Receives a test password message from the user through a Telegram bot created using BotFather.

**2. Edit Fields**

Extracts the password and required Telegram metadata for processing.

**3. Crypto Node**

Generates a SHA-1 hash of the supplied test password.

**4. JavaScript — Hash Splitting**

Splits the 40-character hexadecimal SHA-1 hash into:

- **Prefix:** First 5 hexadecimal characters.
- **Suffix:** Remaining 35 hexadecimal characters.

**5. HTTP Request — HIBP API**

Sends the five-character prefix to the Pwned Passwords Range API.

API endpoint:

`https://api.pwnedpasswords.com/range/{first-5-hash-characters}`

**6. JavaScript — Result Comparison**

Compares the locally retained hash suffix with the suffixes returned by the API. A matching suffix with a positive occurrence count produces a `FOUND` verdict; otherwise, the workflow produces `NOT_FOUND`, provided the API response was processed successfully.

**7. Edit Fields — Data Minimisation**

Prepares a reduced payload containing the task tag, verdict, severity, occurrence count, and guidance type.

**8. Google Gemini AI Agent**

Generates concise security guidance based on the result.

**9. Telegram Notification**

Sends the generated guidance to the user through Telegram.

## 🛡️ Security and Privacy

### K-Anonymity

The workflow uses the HIBP range-query method. Only the first five characters of the SHA-1 hash are sent to the API, while the remaining suffix is compared locally.

This reduces the information disclosed during the lookup. It does not make the process completely anonymous or eliminate all privacy risks.

### AI Data Minimisation

The intended AI input contains only the information required to produce security recommendations.

The original password and complete hash should never be forwarded to the AI model.

### Responsible Testing

- Use only dummy passwords for demonstrations.
- Never submit real account passwords to the Telegram bot.
- Protect API credentials and Telegram bot tokens.
- Disable or carefully restrict execution-data retention to prevent sensitive inputs from remaining in logs.
- Treat `NOT_FOUND` as an absence of a match in the checked dataset, not proof that a password is secure.

## 🧰 Technology Stack

| Technology        | Purpose                             |
| ----------------- | ----------------------------------- |
| n8n               | Workflow automation                 |
| Telegram Bot API  | User interaction and notifications  |
| BotFather         | Telegram bot creation               |
| Crypto node       | SHA-1 hashing                       |
| JavaScript        | Hash splitting and local comparison |
| HTTP Request node | HIBP API integration                |
| Have I Been Pwned | Password exposure lookup            |
| Google Gemini AI  | Security guidance generation        |

## 🚀 Setup and Usage

### Prerequisites

- An n8n instance, either self-hosted or cloud-hosted.
- A Telegram bot token from BotFather.
- A Google Gemini API key with access to a supported model.
- Internet access to the HIBP Pwned Passwords API.

### Installation

1. Clone or download this repository.
2. Open n8n and create a new workflow.
3. Import the `NGP-013 (1).json` workflow file.
4. Configure your Telegram API credentials.
5. Configure your Google Gemini API credentials.
6. Verify the selected Gemini model is available to your API key.
7. Inspect the HTTP Request node and verify its response format.
8. Test the workflow with dummy passwords.
9. Confirm the API response is parsed correctly and the Telegram notification is delivered.

**Important:** Imported workflow JSON may contain credential references specific to the original n8n instance. Configure your own credentials after importing.

### Test Inputs

The original workflow is intended for dummy-password testing, including:

- `Ngp@013` — NGP-013 demonstration input.
- `TestOnly#4821` — additional dummy input.
- `DummyPass!7306` — additional dummy input.

These examples are for controlled demonstrations only and should not be used as real account credentials.

## 📊 Results

The workflow is designed to produce these outcomes:

| Verdict     | Meaning                                                                |
| ----------- | ---------------------------------------------------------------------- |
| `FOUND`     | A matching suffix with a positive occurrence count was found.          |
| `NOT_FOUND` | No matching positive-count suffix was found in the processed response. |
| `ERROR`     | The check could not be completed successfully.                         |

The current uploaded workflow maps `FOUND` to `HIGH` severity and `NOT_FOUND` to `INFO`. API failures should be handled separately rather than classified as `NOT_FOUND`.
