Это платформа на [[React.js]] для полнофункциональных веб приложений. Использование компонентов [[React.js]] для создания пользовательских интерфейсов. Для работы с Next.js нужна [[nodejs]].


# создание приложения
введи в директории где ты будешь работать следуйщую команду перед этим нужно установить [[nodejs]]
```bash
npx create-next-app@latest
```
далее нужно выбрать конфигурацию проэкта
```
What is your project named? my-app
Would you like to use TypeScript? No / Yes
Would you like to use ESLint? No / Yes
Would you like to use Tailwind CSS? No / Yes
Would you like to use `src/` directory? No / Yes
Would you like to use App Router? (recommended) No / Yes
Would you like to customize the default import alias (@/*)? No / Yes
What import alias would you like configured? @/*
```

- `What is your project named?` какое имя будет у твоего проэкта
- `Would you like to use TypeScript?` будешь ли ты использовать в проекте TypeScript
- `Would you like to use ESLint?`это инструмент для статического анализа кода, который помогает находить и исправлять проблемы в JavaScript коде
- `Would you like to use Tailwind CSS?` это utility-first CSS фреймворк для быстрого создания пользовательских интерфейсов.
- `Would you like to use src/ directory?` прашивает, хотите ли вы использовать директорию `src/` для исходного кода вашего проекта.
- `Would you like to use App Router? (recommended) No / Yes` App Router - это новая система маршрутизации в Next.js, основанная на файловой системе.
- `Would you like to customize the default import alias (@/*)? No / Yes` Спрашивает, хотите ли вы настроить псевдоним импорта по умолчанию.
- `What import alias would you like configured? @/` Если вы выбрали "Yes" в предыдущем вопросе, здесь вы можете указать желаемый псевдоним импорта.

После создания проэкта его можно запустить
```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```
и подключиться на сервер