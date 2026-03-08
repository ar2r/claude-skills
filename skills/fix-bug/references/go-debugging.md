# Go Debugging

Открывай этот файл, когда кодовая база написана на Go или симптом похож на
`panic`, race condition, зависание, `context deadline exceeded`, flaky `go test`
или дефект на HTTP/service boundary.

## С чего начать

- Сначала локализуй пакет, handler, use case или конкретный тест. Не начинай с
  широкого `go test ./...`, если уже можно сузить область поиска.
- Предпочитай минимальный воспроизводимый запуск:
  - `go test ./path/to/pkg -run TestName -count=1`
  - `go test ./path/to/pkg -run TestName/Subtest -count=1 -v`
  - `go test ./path/to/pkg -race`
- Для flaky-багов отключай кеш тестов через `-count=1`. Если подозреваешь
  зависимость от порядка, попробуй `-shuffle=on`.
- Если баг проявляется в сервисе или CLI, сначала найди пакет с логикой сбоя,
  потом расширяй проверку до e2e или запуска бинаря.

## Симптомы и первые проверки

| Симптом | Что проверить первым | Как подтвердить |
| --- | --- | --- |
| `panic: invalid memory address` | Инициализацию, `nil` receiver, zero value struct, typed-nil interface | Показать путь, где `nil` или пустое значение попало в use site |
| `concurrent map read/write`, race, flaky test | Shared map/slice/state, захват переменных в goroutine, отсутствие синхронизации | Подтвердить через `go test -race` или детерминированный harness |
| Зависание, deadlock, тест не завершается | Каналы, `WaitGroup`, блокирующий I/O, отсутствие cancel/close | Найти конкретную goroutine или ожидание, которое не освобождается |
| `context deadline exceeded` | Propagation `context`, timeout-цепочку, retry loop, медленную зависимость | Показать, где deadline теряется или какой шаг реально медленный |
| HTTP 4xx/5xx, mismatch на API границе | JSON tags, zero values, middleware, request parsing, status mapping | Узкий `httptest` или contract scenario на boundary |
| Неверные данные в цикле/срезе | Pointer/value semantics, reuse переменных в цикле, мутацию shared state | Тест на конкретную трансформацию или loop edge case |

## Частые Go-ловушки

- Не прикрывай проблему общим `if x == nil`, если контракт не допускает
  отсутствия значения. Сначала выясни, почему значение не было создано.
- Проверяй difference между zero value и "значение действительно отсутствует".
- Для `error` и интерфейсов учитывай typed-nil: интерфейс может быть не `nil`,
  хотя конкретный pointer внутри `nil`.
- Для конкурентных багов подозревай:
  - shared mutable state без lock/ownership;
  - горутины, переживающие lifetime теста или запроса;
  - проверки через `time.Sleep` вместо синхронизации;
  - неявный захват переменных цикла.
- Для интеграций проверяй сериализацию, `context`, timeouts и cleanup соединений
  раньше, чем переписывать бизнес-логику.

## Как чинить и проверять

- Делай минимальный fix в точке сломанного инварианта: создание значения,
  корректная синхронизация, propagation `context`, проверка контракта на
  boundary.
- Для edge cases предпочитай table-driven tests и `t.Run`.
- Для HTTP-дефектов предпочитай `httptest` или узкий contract test вместо
  полного e2e, если этого достаточно.
- Для конкурентных багов запускай `go test -race` по целевому пакету. Если race
  не воспроизводится стабильно, явно помечай validation gap.
- Для perf/CPU проблем не гадай. Используй измеримый сценарий, а `pprof` или
  benchmark подключай только если это реально нужно для локализации.
- Если есть локальный reproducible scenario, но поток состояния неясен,
  отладчик вроде `dlv` допустим как вспомогательный инструмент. Сначала все
  равно стремись к тесту или команде, которую можно повторить.
