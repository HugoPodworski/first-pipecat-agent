# First Pipecat Phone Bot (Twilio)

Learn how to connect your Pipecat bot to a phone number so users can call and have voice conversations. This example shows the complete setup for telephone-based AI interactions using Twilio's telephony services. At the end, you'll be able to talk to your bot on the phone.

## Prerequisites

- Python 3.10+
- [ngrok](https://ngrok.com/docs/getting-started/) (for tunneling)
- [Twilio Account](https://www.twilio.com/login) and [phone number](https://help.twilio.com/articles/223135247-How-to-Search-for-and-Buy-a-Twilio-Phone-Number-from-Console)
- AI Service API keys for: [Deepgram](https://console.deepgram.com/signup), [OpenAI](https://auth.openai.com/create-account), and [Cartesia](https://play.cartesia.ai/sign-up)

## Setup

This example requires running both a server and ngrok tunnel in **two separate terminal windows**.

### Clone this repository

```bash
git clone https://github.com/HugoPodworski/first-pipecat-agent.git
cd first-pipecat-agent
```

### Terminal 1: Start ngrok and Configure Twilio

1. Start ngrok:

   In a new terminal, start ngrok to tunnel the local server:

   ```bash
   ngrok http 7860
   ```

   > Want a fixed ngrok URL? Use the `--subdomain` flag:
   > `ngrok http --subdomain=your_ngrok_name 7860`

2. Update the Twilio Webhook:

   - Go to your Twilio phone number's configuration page
   - Under "Voice Configuration", in the "A call comes in" section:
     - Select "Webhook" from the dropdown
     - Enter your ngrok URL: `https://your-ngrok-url.ngrok.io`
     - Ensure "HTTP POST" is selected
   - Click Save at the bottom of the page
 
### Terminal 2: Server Setup

1. Create `.env` and add your API keys

   ```bash
   cp env.example .env
   # Edit .env and fill in:
   # DEEPGRAM_API_KEY=...
   # CARTESIA_API_KEY=...
   # CEREBRAS_API_KEY=...   # or OPENAI_API_KEY if you switch models
   # TWILIO_ACCOUNT_SID=... # optional but recommended
   # TWILIO_AUTH_TOKEN=...  # optional but recommended
   ```

2. Create a virtual environment and install dependencies

   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Configure ngrok + Twilio (guided helper)

   ```bash
   python setup_ngrok_twilio.py
   ```

   - Pick the Twilio number you want to use
   - The script starts ngrok and prints the public URL
   - It updates your Twilio number to POST to `https://<ngrok-host>/`
   - It optionally writes `PIPECAT_PROXY_HOST=<ngrok-host>` to `.env`

4. Run the bot using the printed ngrok host

   ```bash
   python bot.py --transport twilio --proxy <ngrok-host>
   ```

   > Using `uv`? Run using: `uv run bot.py --transport twilio --proxy <ngrok-host>`

   > 💡 First run note: The initial startup may take ~15 seconds as Pipecat downloads required models, like the Silero VAD model.

### Test Your Phone Bot

**Call your Twilio phone number** to start talking with your AI bot! 🚀

> 💡 **Tip**: Check your server terminal for debug logs showing Pipecat's internal workings.

## Troubleshooting

- **Call doesn't connect**: Verify your ngrok URL is correctly set in the Twilio webhook
- **No audio or bot doesn't respond**: Check that all API keys are correctly set in your `.env` file
- **Webhook errors**: Ensure your server is running and ngrok tunnel is active before making calls
- **ngrok tunnel issues**: Free ngrok URLs change each restart - remember to update Twilio

## Understanding the Call Flow

1. **Incoming Call**: User dials your Twilio number
2. **Webhook**: Twilio sends call data to your ngrok URL
3. **WebSocket**: Your server establishes real-time audio connection via Websocket and exchanges Media Streams with Twilio
4. **Processing**: Audio flows through your Pipecat Pipeline
5. **Response**: Synthesized speech streams back to caller

## Next Steps

- **Deploy to production**: Replace ngrok with a proper server deployment
- **Explore other telephony providers**: Try [Telnyx](https://github.com/pipecat-ai/pipecat-examples/tree/main/telnyx-chatbot) or [Plivo](https://github.com/pipecat-ai/pipecat-examples/tree/main/plivo-chatbot) examples
- **Advanced telephony features**: Check out [pipecat-examples](https://github.com/pipecat-ai/pipecat-examples) for call recording, transfer, and more
- **Join Discord**: Connect with other developers on [Discord](https://discord.gg/pipecat)

## Outbound calls (Twilio → phone)

You can also place outbound calls that connect the callee to your running Pipecat bot.

Prereqs:
- Your server must be running locally: `python bot.py --transport twilio --proxy your_ngrok.ngrok.io`
- Your ngrok tunnel must be active and public (same host you pass via `--proxy`)
- `TWILIO_ACCOUNT_SID` and `TWILIO_AUTH_TOKEN` set in your environment
- A Twilio phone number to place calls from

Make the call:

```bash
python outbound.py --to +15551234567 --from +15557654321 --proxy your_ngrok.ngrok.io
```

Notes:
- The script sends Twilio to `https://<proxy>/` with HTTP POST. The Pipecat runner responds with the XML that instructs Twilio to open a Media Streams WebSocket to `wss://<proxy>/ws`.
- You can override the webhook URL directly with `--url https://example.com/` if you host the runner elsewhere.

## Auto-configure Twilio + ngrok

Use this helper to:
- Start an ngrok tunnel
- Pick which Twilio number to configure
- Set "A call comes in" webhook to your ngrok URL (HTTP POST)
- Optionally set "Primary handler fails" to the same URL
- Optionally write `PIPECAT_PROXY_HOST` to `.env`

```bash
# Persistent tunnel by default; choose number interactively
python setup_ngrok_twilio.py

# Or pick a specific number and auto-launch the bot
python setup_ngrok_twilio.py --to +15551234567 --launch-bot

# If you only want to update Twilio and exit without keeping ngrok running:
python setup_ngrok_twilio.py --no-stay-running
```

Environment:
- `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` required
- `NGROK_AUTHTOKEN` recommended for reliability
- `NGROK_REGION` optional (e.g. `us`)

After it prints the public URL, start the bot in another terminal (if you didn't use --launch-bot):

```bash
python bot.py --transport twilio --proxy <ngrok-host>
```