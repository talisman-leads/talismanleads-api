# TalismanLeads API

Run the TalismanLeads bulk phone, email and carrier checkers from your own server. Send a list, see the price, place the order and download the result, paid from the same balance and at the same prices as the [website](https://talismanleads.io).

[![API docs](https://img.shields.io/badge/API%20docs-docs.talismanleads.io-0d9488)](https://docs.talismanleads.io)
[![OpenAPI](https://img.shields.io/badge/OpenAPI-3.1-0d9488)](openapi.json)
[![Checkers](https://img.shields.io/badge/checkers-49-0d9488)](https://docs.talismanleads.io/checkers)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

- **Base address:** `https://api.talismanleads.io/v1`
- **Format:** JSON in, JSON out. [OpenAPI 3.1 file](openapi.json) (also at [https://docs.talismanleads.io/openapi.json](https://docs.talismanleads.io/openapi.json))
- **Authentication:** `Authorization: Bearer <key>` (or `X-API-Key: <key>`)

## Quick start

1. Create an API key under [API keys](https://talismanleads.io/api-keys) in your TalismanLeads account, and add a balance.
2. Keep the key in an environment variable:

   ```bash
   export TALISMANLEADS_API_KEY="tl_live_your_key_here"
   ```

3. Check your balance:

   ```bash
   curl "https://api.talismanleads.io/v1/account" -H "Authorization: Bearer $TALISMANLEADS_API_KEY"
   ```

4. Run any checker over a list. The example shows the price and asks before it orders:

   ```bash
   python3 examples/python/check.py my-list.txt --checker whatsapp_status_detection
   # or
   node examples/node/check.mjs my-list.txt --checker whatsapp_status_detection
   ```

The examples need nothing installed beyond Python 3.8+ or Node.js 18.3+. They upload large lists, retry safely, wait for the order and save the result files in `results/`.

## Endpoints

| Call | What it does | Limit |
| --- | --- | --- |
| `GET /account` | Your balance and the limits in force | 60 a minute |
| `GET /checkers` | The checkers your account can order, at your prices | 60 a minute |
| `POST /uploads` | A link to upload a large list to (up to 200 MB, 10,000,000 entries) | 10 a minute |
| `POST /estimates` | What an order would cost. Charges nothing | 10 a minute |
| `POST /orders` | Place an order. Needs an `Idempotency-Key` | 10 a minute |
| `GET /orders` | Your orders | 60 a minute |
| `GET /orders/{id}` | One order and its status | 60 a minute |
| `GET /orders/{id}/results` | Download links for the result files | 30 a minute |

The full reference, with every parameter and answer, is at [https://docs.talismanleads.io/reference](https://docs.talismanleads.io/reference). Limits are per customer across all keys; see [Rate limits](https://docs.talismanleads.io/limits).

## The order flow

```text
POST /estimates  ->  POST /orders  ->  GET /orders/{id} every 30 s  ->  GET /orders/{id}/results
   (price)           (charge)          queued, processing,                (download links,
                                       completed / partially_completed     good for 15 minutes)
                                       / failed
```

- **Entries:** up to 100,000 in the request as `entries`. Larger lists go through `POST /uploads` and are sent as `upload_id`.
- **Phone numbers** are written with their country code (`+14155550142`), and at least 90% of one file should be from one country.
- **Minimum:** every file is billed as at least the checker's minimum, so `billed_entries` can be more than `entries`.
- **Refunds:** parts that fail are refunded; `refunded` on the order shows what came back.
- **Statuses:** treat a status you do not know as still running. More may be added.

## Safe retries

`POST /orders` needs an `Idempotency-Key` header: up to 200 ASCII characters with no spaces, such as a UUID. For 7 days the same key means the same order, charged once. When a request times out or answers `order_status_unknown` or a gateway 502/504, wait a minute and send it again with the **same** key. A new key is a new order, and is charged. See [Safe retries](https://docs.talismanleads.io/retries).

## Errors

```json
{
  "error": {
    "code": "invalid_key",
    "message": "That API key is not valid.",
    "request_id": "req_ea3cd7baa97f46e399d1e59f99f57935",
    "docs_url": "https://docs.talismanleads.io/errors#invalid_key"
  }
}
```

Act on `code`, which never changes; `message` is for people and may change. Every code is listed in [Errors](https://docs.talismanleads.io/errors).

## Checkers

Each checker has its own repo with a ready-to-run example.

### Messaging apps

| Checker | Takes | Repo |
| --- | --- | --- |
| [iMessage Number Checker](https://docs.talismanleads.io/checkers/imessage_number_checker) | Phone numbers | [imessage-number-checker-api](https://github.com/talisman-leads/imessage-number-checker-api) |
| [MAX Number Checker](https://docs.talismanleads.io/checkers/max_number_checker) | Phone numbers | [max-number-checker-api](https://github.com/talisman-leads/max-number-checker-api) |
| [MAX Profile Checker](https://docs.talismanleads.io/checkers/max_profile_checker) | Phone numbers | [max-profile-checker-api](https://github.com/talisman-leads/max-profile-checker-api) |
| [Messenger Number Checker](https://docs.talismanleads.io/checkers/messenger_number_checker) | Phone numbers | [messenger-number-checker-api](https://github.com/talisman-leads/messenger-number-checker-api) |
| [RCS Number Checker](https://docs.talismanleads.io/checkers/rcs_number_checker) | Phone numbers | [rcs-number-checker-api](https://github.com/talisman-leads/rcs-number-checker-api) |
| [Telegram Activity Checker](https://docs.talismanleads.io/checkers/telegram_activity_checker) | Phone numbers | [telegram-activity-checker-api](https://github.com/talisman-leads/telegram-activity-checker-api) |
| [Telegram Number Checker](https://docs.talismanleads.io/checkers/telegram_number_checker) | Phone numbers | [telegram-number-checker-api](https://github.com/talisman-leads/telegram-number-checker-api) |
| [Telegram Profile Checker](https://docs.talismanleads.io/checkers/telegram_profile_checker) | Phone numbers | [telegram-profile-checker-api](https://github.com/talisman-leads/telegram-profile-checker-api) |
| [Viber Activity Checker](https://docs.talismanleads.io/checkers/viber_activity_checker) | Phone numbers | [viber-activity-checker-api](https://github.com/talisman-leads/viber-activity-checker-api) |
| [Viber Number Checker](https://docs.talismanleads.io/checkers/viber_number_checker) | Phone numbers | [viber-number-checker-api](https://github.com/talisman-leads/viber-number-checker-api) |
| [WhatsApp Business Checker](https://docs.talismanleads.io/checkers/whatsapp_business_checker) | Phone numbers | [whatsapp-business-checker-api](https://github.com/talisman-leads/whatsapp-business-checker-api) |
| [WhatsApp Number Checker](https://docs.talismanleads.io/checkers/whatsapp_status_detection) | Phone numbers | [whatsapp-number-checker-api](https://github.com/talisman-leads/whatsapp-number-checker-api) |
| [WhatsApp Profile Picture, Age, Ethnicity Checker](https://docs.talismanleads.io/checkers/whatsapp_profile_checker) | Phone numbers | [whatsapp-profile-checker-api](https://github.com/talisman-leads/whatsapp-profile-checker-api) |

### Social networks

| Checker | Takes | Repo |
| --- | --- | --- |
| [Facebook Email Checker](https://docs.talismanleads.io/checkers/facebook_email_checker) | Email addresses | [facebook-email-checker-api](https://github.com/talisman-leads/facebook-email-checker-api) |
| [Facebook Number Checker](https://docs.talismanleads.io/checkers/facebook_number_checker) | Phone numbers | [facebook-number-checker-api](https://github.com/talisman-leads/facebook-number-checker-api) |
| [Instagram Email Checker](https://docs.talismanleads.io/checkers/instagram_email_checker) | Email addresses | [instagram-email-checker-api](https://github.com/talisman-leads/instagram-email-checker-api) |
| [Instagram Number Checker](https://docs.talismanleads.io/checkers/instagram_number_checker) | Phone numbers | [instagram-number-checker-api](https://github.com/talisman-leads/instagram-number-checker-api) |
| [LinkedIn Email Checker](https://docs.talismanleads.io/checkers/linkedin_email_checker) | Email addresses | [linkedin-email-checker-api](https://github.com/talisman-leads/linkedin-email-checker-api) |
| [LinkedIn Number Checker](https://docs.talismanleads.io/checkers/linkedin_number_checker) | Phone numbers | [linkedin-number-checker-api](https://github.com/talisman-leads/linkedin-number-checker-api) |
| [LinkedIn Profile, Gender, Industry, Job Title, Salary Checker](https://docs.talismanleads.io/checkers/linkedin_profile_checker) | Email addresses | [linkedin-profile-checker-api](https://github.com/talisman-leads/linkedin-profile-checker-api) |
| [Snapchat Number Checker](https://docs.talismanleads.io/checkers/snapchat_number_checker) | Phone numbers | [snapchat-number-checker-api](https://github.com/talisman-leads/snapchat-number-checker-api) |
| [Threads Number Checker](https://docs.talismanleads.io/checkers/threads_number_checker) | Phone numbers | [threads-number-checker-api](https://github.com/talisman-leads/threads-number-checker-api) |
| [TikTok Number Checker](https://docs.talismanleads.io/checkers/tiktok_number_checker) | Phone numbers | [tiktok-number-checker-api](https://github.com/talisman-leads/tiktok-number-checker-api) |
| [VK Number Checker](https://docs.talismanleads.io/checkers/vk_number_checker) | Phone numbers | [vk-number-checker-api](https://github.com/talisman-leads/vk-number-checker-api) |

### Shopping, streaming and services

| Checker | Takes | Repo |
| --- | --- | --- |
| [Airbnb Number Checker](https://docs.talismanleads.io/checkers/airbnb_number_checker) | Phone numbers | [airbnb-number-checker-api](https://github.com/talisman-leads/airbnb-number-checker-api) |
| [Amazon Email Checker](https://docs.talismanleads.io/checkers/amazon_email_checker) | Email addresses | [amazon-email-checker-api](https://github.com/talisman-leads/amazon-email-checker-api) |
| [Amazon Number Checker](https://docs.talismanleads.io/checkers/amazon_number_checker) | Phone numbers | [amazon-number-checker-api](https://github.com/talisman-leads/amazon-number-checker-api) |
| [Apple ID Email Checker](https://docs.talismanleads.io/checkers/apple_id_email_checker) | Email addresses | [apple-id-email-checker-api](https://github.com/talisman-leads/apple-id-email-checker-api) |
| [Apple ID Number Checker](https://docs.talismanleads.io/checkers/apple_id_number_checker) | Phone numbers | [apple-id-number-checker-api](https://github.com/talisman-leads/apple-id-number-checker-api) |
| [DHL Number Checker](https://docs.talismanleads.io/checkers/dhl_number_checker) | Phone numbers | [dhl-number-checker-api](https://github.com/talisman-leads/dhl-number-checker-api) |
| [Gmail Email Checker](https://docs.talismanleads.io/checkers/gmail_email_checker) | Email addresses | [gmail-email-checker-api](https://github.com/talisman-leads/gmail-email-checker-api) |
| [Microsoft Number Checker](https://docs.talismanleads.io/checkers/microsoft_number_checker) | Phone numbers | [microsoft-number-checker-api](https://github.com/talisman-leads/microsoft-number-checker-api) |
| [Netflix Email Checker](https://docs.talismanleads.io/checkers/netflix_email_checker) | Email addresses | [netflix-email-checker-api](https://github.com/talisman-leads/netflix-email-checker-api) |
| [Netflix Number Checker](https://docs.talismanleads.io/checkers/netflix_number_checker) | Phone numbers | [netflix-number-checker-api](https://github.com/talisman-leads/netflix-number-checker-api) |
| [PayPal Email Checker](https://docs.talismanleads.io/checkers/paypal_email_checker) | Email addresses | [paypal-email-checker-api](https://github.com/talisman-leads/paypal-email-checker-api) |
| [Spotify Email Checker](https://docs.talismanleads.io/checkers/spotify_email_checker) | Email addresses | [spotify-email-checker-api](https://github.com/talisman-leads/spotify-email-checker-api) |
| [Temu Number Checker](https://docs.talismanleads.io/checkers/temu_number_checker) | Phone numbers | [temu-number-checker-api](https://github.com/talisman-leads/temu-number-checker-api) |
| [Wheely Number Checker](https://docs.talismanleads.io/checkers/wheely_number_checker) | Phone numbers | [wheely-number-checker-api](https://github.com/talisman-leads/wheely-number-checker-api) |

### Crypto exchanges

| Checker | Takes | Repo |
| --- | --- | --- |
| [Binance Number Checker](https://docs.talismanleads.io/checkers/binance_number_checker) | Phone numbers | [binance-number-checker-api](https://github.com/talisman-leads/binance-number-checker-api) |
| [CoinW Email Checker](https://docs.talismanleads.io/checkers/coinw_email_checker) | Email addresses | [coinw-email-checker-api](https://github.com/talisman-leads/coinw-email-checker-api) |
| [CoinW Number Checker](https://docs.talismanleads.io/checkers/coinw_number_checker) | Phone numbers | [coinw-number-checker-api](https://github.com/talisman-leads/coinw-number-checker-api) |
| [Crypto.com Email Checker](https://docs.talismanleads.io/checkers/crypto_com_email_checker) | Email addresses | [crypto-com-email-checker-api](https://github.com/talisman-leads/crypto-com-email-checker-api) |
| [Crypto.com Number Checker](https://docs.talismanleads.io/checkers/crypto_com_number_checker) | Phone numbers | [crypto-com-number-checker-api](https://github.com/talisman-leads/crypto-com-number-checker-api) |
| [HTX Email Checker](https://docs.talismanleads.io/checkers/htx_email_checker) | Email addresses | [htx-email-checker-api](https://github.com/talisman-leads/htx-email-checker-api) |
| [HTX Number Checker](https://docs.talismanleads.io/checkers/htx_number_checker) | Phone numbers | [htx-number-checker-api](https://github.com/talisman-leads/htx-number-checker-api) |
| [KuCoin Email Checker](https://docs.talismanleads.io/checkers/kucoin_email_checker) | Email addresses | [kucoin-email-checker-api](https://github.com/talisman-leads/kucoin-email-checker-api) |
| [KuCoin Number Checker](https://docs.talismanleads.io/checkers/kucoin_number_checker) | Phone numbers | [kucoin-number-checker-api](https://github.com/talisman-leads/kucoin-number-checker-api) |
| [OKX Number Checker](https://docs.talismanleads.io/checkers/okx_number_checker) | Phone numbers | [okx-number-checker-api](https://github.com/talisman-leads/okx-number-checker-api) |

### Carrier and line type

| Checker | Takes | Repo |
| --- | --- | --- |
| [Number Carrier Lookup](https://docs.talismanleads.io/checkers/global_carrier_checker) | Phone numbers | [phone-carrier-lookup-api](https://github.com/talisman-leads/phone-carrier-lookup-api) |

## Keep your key safe

- Use the key only on your server. The API does not accept calls from a web page.
- Keep it out of your code and version control: this repo's `.gitignore` already leaves out `.env`.
- Upload and download links carry their own permission. Never send your key to them.
- If a key may have been seen, revoke it in your account at once.

## Support

[support@talismanleads.io](mailto:support@talismanleads.io), with the `request_id` from the answer. Issues are off on this repo, so that every question reaches the support team.

## License

The example code is under the [MIT License](LICENSE). Use of the API is under the [API terms](https://docs.talismanleads.io/terms) and the [Terms of Service](https://talismanleads.io/terms).
