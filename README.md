# recipes

Flex-рецепти пакетів Laba: коли сервіс встановлює бандл, Flex реєструє його в `config/bundles.php`
і створює `config/packages/<alias>.yaml` з рецепта.

Сервіси підключають репозиторій у `composer.json` (так уже зроблено в `template-symfony-service`):

```json
{
    "extra": {
        "symfony": {
            "endpoint": [
                "https://raw.githubusercontent.com/Mettaron/recipes/main/index.json",
                "flex://defaults"
            ]
        }
    }
}
```

## Структура

```
laba/<package>/<version>/          джерело рецепта — правиться руками
├── manifest.json                  що робить Flex: bundles, copy-from-recipe, env, ...
└── config/packages/<alias>.yaml   файли, які копіюються в сервіс
r/laba/<package>/<version>.json    скомпільований рецепт — те, що читає Flex (генерується)
index.json                         список рецептів і посилання для Flex (recipes — генерується)
bin/compile                        laba/ → r/ і index.json
docker/Dockerfile                  nginx з index.json, r/ і laba/ — щоб роздавати рецепти не з GitHub
```

## Версії

Flex бере рецепт найбільшої версії, не вищої за версію встановленого пакета: рецепт `0.1` діє і для
пакета `0.3`, поки не з'явиться рецепт `0.2` чи `0.3`. Тому нова версія рецепта потрібна лише тоді,
коли змінюється те, що він створює (нові опції конфігу, env-змінні), — а не на кожен реліз пакета.

| Пакет | Рецепти | Що змінилось |
|---|---|---|
| `laba/auth-client` | 0.1 | |
| `laba/brute-force-bundle` | 0.1 | сховище лічильників на власному Redis сервісу (`REDIS_DSN`), `when@test` — файли й `flock` |
| `laba/client-bundle` | 0.1, 0.2 | 0.2: `global_cache` потребує `symfony/cache`; DSN спільного `laba_redis` |
| `laba/loggable-bundle` | 0.1 | |
| `laba/service-bundle` | 0.1, 0.2, 0.3 | 0.2: опція `api_request_log.entity_manager`; 0.3: розділ `api_access` (ключі сервісів, що викликають) |

Пакети без рецепта (`laba/request-bundle`, `laba/lib-bundle`) Flex реєструє сам (auto-generated recipe):
у них немає обов'язкового конфігу.

## Як додати чи змінити рецепт

1. Нова версія — скопіюй попередню: `cp -r laba/service-bundle/0.2 laba/service-bundle/0.3`.
   Опубліковану версію рецепта не змінюй: сервіси, що вже її встановили, порівнюють з нею (`composer recipes`).
2. Зміни `manifest.json` і файли в `config/`.
3. Згенеруй скомпільовані файли й закоміть разом із джерелом:

   ```bash
   bin/compile          # пише r/ і index.json
   bin/compile --check  # нічого не пише; exit 1, якщо r/ чи index.json застаріли
   ```

4. Перевір, що конфіг із рецепта приймає бандл (його дерево конфігу), — інакше сервіс після
   `composer require` не збереться.

Рецепт клієнта сервісу готується в його репозиторії (`recipe/` з `template-symfony-client-bundle`)
і переноситься сюди як `laba/<name>-client/0.1/`.
