# mobile-skeleton

Expo + Supabase + Stripe starter for the [buildthisniw](https://github.com/ZeiProX76/codekit) kit.

Ships the **revenue-arbitrage** pattern by default: users pay on your web checkout, deep-link back into the app, and their entitlement flips on. Skip Apple's 30% where App Review rules allow it.

## Stack

- Expo SDK 54 + Expo Router v6
- Supabase (auth, Postgres, RLS, edge functions)
- Stripe Checkout on web + deep-link return into Expo
- NativeWind v4
- EAS Build + Submit

## Usage

Clone via the `/skeleton mobile` command inside a buildthisniw-initialized project. Or clone directly:

```bash
git clone https://github.com/ZeiProX76/mobile-skeleton.git my-app
cd my-app
npm install
```

## Status

Baseline only. Scaffold lands in the next commit.

## License

MIT — see [LICENSE](LICENSE).
