# Complyx - IFRS S1 & S2 Readiness Assessment Chatbot

Complyx is a conversational AI assistant designed to assess organizational readiness for IFRS S1 (General Requirements for Disclosure of Sustainability-related Financial Information) and IFRS S2 (Climate-related Disclosures).

## Project Structure

```
ifrsbot/
├── client/          # Next.js frontend (Vercel)
├── server/          # Node.js/Express backend (Railway)
│   ├── docker/      # Docker services for local development
│   └── prisma/      # Database schema and migrations
└── roadmap.md       # Project roadmap and documentation
```

## Getting Started

### Prerequisites

- Node.js >= 18.0.0
- pnpm >= 8.0.0
- Docker (for local development services)

### Installation

1. Install dependencies:
```bash
pnpm install
cd client && pnpm install
cd ../server && pnpm install
```

2. Set up environment variables:
```bash
# Copy example files
cp .env.example .env
cp client/.env.example client/.env.local
cp server/.env.example server/.env
```

   **Important**: Add your Google Gemini API key to `server/.env`:
   ```env
   GEMINI_API_KEY=your_gemini_api_key_here
   AI_PROVIDER=gemini
   ```
   
   Get your Gemini API key: https://makersuite.google.com/app/apikey

3. Start Docker services (PostgreSQL and Redis):
```bash
cd server/docker
docker-compose up -d
```

4. Set up database:
```bash
cd server
pnpm db:generate
pnpm db:migrate
```

5. Start development servers:
```bash
# Terminal 1: Start client
cd client
pnpm dev

# Terminal 2: Start server
cd server
pnpm dev
```

## Development

- Client runs on: http://localhost:3000
- Server runs on: http://localhost:3001
- PostgreSQL: localhost:5432
- Redis: localhost:6379

## Tech Stack

- **Frontend**: Next.js 14, TypeScript, Tailwind CSS
- **Backend**: Node.js, Express, TypeScript
- **Database**: PostgreSQL (Prisma ORM)
- **Cache**: Redis
- **AI**: Google Gemini API (default), Multi-provider support (Grok, OpenAI)
- **Storage**: Cloudflare R2 (for media and documents)

## AI Provider Configuration

The application supports multiple AI providers with Google Gemini as the default:

- **Default Provider**: Google Gemini API (free tier: 60 req/min, 1,500 req/day)
- **Future Providers**: Grok API, OpenAI GPT-4 (can be added via environment variables)

### Getting AI API Key

**Google Gemini (Recommended for Development)**:
1. Visit [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Create a new API key
4. Add to `server/.env`: `GEMINI_API_KEY=your_key_here`

**Switching Providers**:
- Set `AI_PROVIDER=gemini` (default) for Google Gemini
- Set `AI_PROVIDER=grok` for Grok API (when available)
- Set `AI_PROVIDER=openai` for OpenAI (when configured)

### Multi-Provider Architecture

The application uses a provider-agnostic architecture:
- Easy switching between providers via environment variables
- No client-side changes required
- Fallback support for reliability
- Cost optimization based on use case

## Project Status

**Week 1: Project Setup and Architecture** ✅
- Repository and structure setup
- Client (Next.js) setup
- Server (Express) setup
- Docker services configuration
- Database schema design

**Week 2: Core Chat Interface** ✅
- Chat interface components
- Message bubbles and input
- Typing indicators
- State management with Zustand

## License

Private - All Rights Reserved
