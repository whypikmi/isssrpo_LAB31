# Лабораторная работа №31-32. Введение в SQLite и Entity Framework Core

### Информация
**Студент:** Телятникова Е.П.

**Группа:** ИСП-231

### Краткое описание работы
В данной лабораторной работе было разработано веб-API на ASP.NET Core с использованием Entity Framework Core и SQLite. Реализован Code First подход — модели C# автоматически создают структуру базы данных через миграции. API предоставляет полный набор CRUD операций для управления списком задач, а также расширенные возможности: поиск, фильтрацию, пагинацию, статистику и просмотр просроченных задач.

**Основные изученные технологии:**
- Entity Framework Core (ORM)
- SQLite (встраиваемая БД)
- LINQ (Language Integrated Query)
- Миграции EF Core
- Swagger/OpenAPI документация

### Полезные команды dotnet ef

| Команда | Описание |
|---|---|
| `dotnet ef migrations add Имя` | Создать новую миграцию на основе изменений в моделях |
| `dotnet ef database update` | Применить все неприменённые миграции к БД |
| `dotnet ef migrations list` | Показать список миграций и их статус |
| `dotnet ef migrations remove` | Удалить последнюю миграцию (если не применена) |
| `dotnet ef database update ПредыдущаяМиграция` | Откатить до указанной миграции |
| `dotnet ef migrations script` | Сгенерировать SQL-скрипт миграции |
| `dotnet ef database update --verbose` | Применить миграции с подробным выводом SQL |

### Структура проекта
```
Lab31-32_EFCore/
├── TaskDb/
│   ├── Controllers/
│   │   └── TasksController.cs          
│   ├── Data/
│   │   └── AppDbContext.cs             
│   ├── Models/
│   │   ├── TaskItem.cs                 
│   │   └── TaskDto.cs                 
│   ├── Migrations/                     
│   │   ├── *InitialCreate.cs
│   │   └── *AddDueDateToTask.cs
│   ├── Properties/
│   │   └── launchSettings.json
│   ├── appsettings.json                
│   ├── appsettings.Development.json    
│   ├── Program.cs                     
│   └── TaskDb.csproj                   
├── img/                                
│   ├── gitPushLab31-32_Telyatnikova.png
│   ├── step4_migrationLab31-32_Telyatnikova.png
│   ├── step5_crudLab31-32_Telyatnikova.png
│   ├── step6_linqLab32_Telyatnikova.png
│   ├── step7_migrationLab32_Telyatnikova.png
│   └── step8_sqlLogsLab32_Telyatnikova.png
├── .editorconfig
├── .gitignore
└── README.md
```
### Список реализованных маршрутов
| Метод | Маршрут | Описание |
|---|---|---|
| GET | `/api/tasks` | Получить все задачи (с фильтрацией по `completed` и `priority`) |
| GET | `/api/tasks/{id}` | Получить задачу по ID |
| POST | `/api/tasks` | Создать новую задачу |
| PUT | `/api/tasks/{id}` | Полностью обновить задачу |
| PATCH | `/api/tasks/{id}/complete` | Переключить статус выполнения |
| DELETE | `/api/tasks/{id}` | Удалить задачу |
| GET | `/api/tasks/search` | Поиск по тексту, приоритету и статусу |
| GET | `/api/tasks/stats` | Получить статистику по задачам |
| GET | `/api/tasks/paged` | Пагинация (параметры: `page`, `pageSize`) |
| GET | `/api/tasks/overdue` | Получить просроченные задачи |

### Таблица применённых миграций
| Название миграции | Дата создания | Изменения |
|---|---|---|
| `InitialCreate` | [дата] | Создание таблицы `Tasks` со всеми полями: Id, Title, Description, IsCompleted, CreatedAt, Priority. Добавление seed-данных (3 задачи) |
| `AddDueDateToTask` | [дата] | Добавление поля `DueDate` (DateTime?) в таблицу `Tasks` |

### Сравнительная таблица LINQ vs SQL
| LINQ | SQL |
|---|---|
| `.Where(t => t.IsCompleted == false)` | `WHERE is_completed = 0` |
| `.OrderBy(t => t.CreatedAt)` | `ORDER BY created_at ASC` |
| `.OrderByDescending(t => t.CreatedAt)` | `ORDER BY created_at DESC` |
| `.Take(10)` | `LIMIT 10` |
| `.Skip(20).Take(10)` | `OFFSET 20 LIMIT 10` |
| `.Count()` | `SELECT COUNT(*)` |
| `.Any(t => t.Priority == "High")` | `SELECT EXISTS(... WHERE priority = 'High')` |
| `.Max(t => t.Id)` | `SELECT MAX(id)` |
| `.GroupBy(t => t.Priority)` | `GROUP BY priority` |
| `.Select(t => t.Title)` | `SELECT title` |
| `.Contains("SQL")` | `LIKE '%SQL%'` |

### Итоговая сравнительная таблица
| Концепция | Хранение в памяти | EF Core + SQLite |
|---|---|---|
| **Хранение данных** | `static List<T>` в RAM | Файл `.db` на диске |
| **После перезапуска** | Данные пропадают | Данные сохраняются |
| **Поиск по условию** | LINQ to Objects | LINQ to Entities → SQL |
| **Создание структуры** | Не нужно | Миграции (`dotnet ef`) |
| **Начальные данные** | Хардкод в коде | `HasData()` в миграции |
| **Получение данных** | `list.FirstOrDefault(...)` | `await db.Table.FindAsync(id)` |
| **Добавление** | `list.Add(item)` | `db.Table.Add(item)` + `SaveChangesAsync()` |
| **Удаление** | `list.Remove(item)` | `db.Table.Remove(item)` + `SaveChangesAsync()` |
| **Масштабируемость** | Ограничена RAM | Гигабайты данных |
| **Транзакции** | Нет | Встроены в EF Core |

### Главные выводы
1. **EF Core — это переводчик между C# и SQL.** Вы пишете LINQ-запросы к объектам, а EF Core автоматически преобразует их в оптимизированные SQL-запросы. Это позволяет работать с БД, не покидая экосистему C#.
2. **Миграции — это система контроля версий для структуры БД.** Как Git для кода, миграции позволяют отслеживать, применять и откатывать изменения схемы базы данных, а также синхронизировать их в команде.
3. **Code First удобнее ручного написания SQL.** Изменил класс модели → создал миграцию (`dotnet ef migrations add`) → применил к БД (`dotnet ef database update`). Никаких ручных `ALTER TABLE` и ошибок в синтаксисе.
4. **`SaveChangesAsync()` — ключевой момент.** До вызова этого метода все изменения (Add, Remove, Update) живут только в памяти и в ChangeTracker EF Core. Только после `SaveChangesAsync()` они фиксируются в базе данных.
5. **`async/await` при работе с БД — стандарт, а не опция.** Блокировать поток на время ожидания ответа от БД — плохая практика, которая снижает пропускную способность сервера. Асинхронные методы позволяют потоку обрабатывать другие запросы, пока БД выполняет запрос.