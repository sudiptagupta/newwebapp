# Dark Theme Authentication UI with Quasar

A modern authentication system built with Vue 3, Quasar Framework, and Supabase, featuring a sleek dark theme design.

## Features

- 🌙 Dark theme UI
- 🔐 Complete authentication flow:
  - Login
  - Registration
  - Password Recovery
  - Password Reset
- 📱 Responsive design
- 🎨 Consistent styling across all pages
- ⚡ Real-time validation
- 📧 Email-based password recovery

## Tech Stack

- Vue 3
- Quasar Framework
- Supabase (Authentication)
- SCSS for styling

## Installation

1. Clone the repository:

```bash
git clone [your-repository-url]
```

2. Install dependencies:

```bash
yarn
# or
npm install
```

3. Configure Supabase:
   - Create a Supabase project
   - Copy your Supabase URL and anon key
   - Create a `.env` file with:

```env
SUPABASE_URL=your-supabase-url
SUPABASE_KEY=your-supabase-anon-key
```

4. Start the development server:

```bash
quasar dev
```

## Project Structure

```
src/
├── pages/
│   ├── LoginPage.vue
│   ├── RegisterPage.vue
│   ├── ForgotPasswordPage.vue
│   └── ResetPasswordPage.vue
├── css/
│   └── quasar.variables.scss
├── composables/
│   ├── UserAuthUser.js
│   └── UseNotify.js
└── boot/
    └── supabase.js
```

## Authentication Pages

- **Login**: Email/password authentication
- **Register**: New user registration
- **Forgot Password**: Password recovery request
- **Reset Password**: New password setup

## Styling

The project uses a consistent dark theme with:

- Dark backgrounds
- White text inputs
- Consistent card styling
- Responsive design

Global styles are maintained in `quasar.variables.scss` for consistency.

## Development

### Customize the configuration

See [Configuring quasar.config.js](https://v2.quasar.dev/quasar-cli-webpack/quasar-config-js).

### Lint the files

```bash
yarn lint
# or
npm run lint
```

### Format the files

```bash
yarn format
# or
npm run format
```

## Building for Production

```bash
quasar build
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

[MIT License](LICENSE)

## Acknowledgments

- [Quasar Framework](https://quasar.dev/)
- [Supabase](https://supabase.io/)
- [Vue.js](https://vuejs.org/)
