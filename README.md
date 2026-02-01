# PixSTD.DataUtilities
**Официальная библиотека от PixSTD**

📦 NuGet: `PixSTD.DataUtilities`  
👨💻 Автор: PixSTD  
🐙 Исходный код: https://github.com/PixSTD/DataUtilities.dll  
📄 Лицензия: MIT

**PixSTD.DataUtilities** — продвинутая библиотека для .NET, предоставляющая:

- **🔐 Безопасность** — шифрование и защита данных
- **🗂️ Хранение** — организация файлов с хэшированием путей  
- **📡 Сетевая подготовка** — упаковка/распаковка данных для сетевой передачи
- **📦 Производительность** — быстрая сериализация через MessagePack
- **⚡ Совместимость** — .NET Standard 2.1, 8, 9, 10
- **🌐 Доступность** — Windows, Linux, Android
- **📝 Наблюдаемость** — полное логирование операций

## 📑 Оглавление

- [🚀 Быстрый старт](#-быстрый-старт)
- [📖 Инициализация](#-инициализация)
  - [🔧 Конструкторы](#-конструкторы)
  - [⚙️ Методы настройки](#-методы-настройки-можно-вызывать-после-создания)
  - [🎯 Примеры использования](#-примеры-использования)
  - [❗ Важные замечания](#-важные-замечания)
- [💾 Сохранение и загрузка данных](#-сохранение-и-загрузка-данных)
  - [📤 Сохранение объектов](#-сохранение-объектов)
  - [📥 Загрузка объектов](#-загрузка-объектов)
  - [🔄 Работа с разными типами данных](#-работа-с-разными-типами-данных)
  - [🎯 Пример использования](#-пример-использования)
- [📁 Копирование файлов и папок](#-копирование-файлов-и-папок)
  - [📄 Копирование файлов](#-копирование-файлов)
  - [📂 Копирование папок (рекурсивное)](#-копирование-папок-рекурсивное)
  - [🎯 Пример использования](#-пример-использования-1)
  - [🔍 Поиск файлов и папок](#-поиск-файлов-и-папок)
  - [🗑️ Удаление файлов и папок](#-удаление-файлов-и-папок)
- [🛠️ Вспомогательные функции](#-вспомогательные-функции)
  - [📂 Работа с путями](#-работа-с-путями)
  - [🔄 Конвертация объектов](#-конвертация-объектов)
- [🔐 Шифрование и работа с сетью](#-шифрование-и-работа-с-сетью)
  - [🔒 Локальное шифрование](#-локальное-шифрование)
  - [🌐 Сетевое взаимодействие](#-сетевое-взаимодействие)
- [📝 Логирование и обработка событий](#-логирование-и-обработка-событий)
  - [🎯 Система логирования](#-система-логирования)
  - [🔔 Событие Log](#-событие-log)

## 🚀 Быстрый старт
```csharp
	using DataUtilities;
	using System;
	using System.IO;
	using System.Threading.Tasks;

	// 1. Инициализация
	
	string localAppData = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
	string appFolder = Path.Combine(localAppData, "CompanyName", "YourAppName");
	
	var data = new PlayerData(
		hash: "ваш_ключ_шифрования",			// byte[] или string
		startPath: appFolder,					// куда сохранять файлы
		lengthNameDirectory: HexLength.Short, 	// 8 символов для папки
		lengthNameFile: HexLength.Short,       	// 8 символов для файла (Пример: "player/profile" → "a1b2c3d4/e5f67890")
		offsetHashHex: -1						// Смещение начала хэша. Значение -1 означает автоматический сдвиг, равный длине хэшируемого сегмента
	);
	
	
	// 2. Сохранение данных (асинхронно)
	
	await data.SaveA(profile, "data/player/profile");
	// → сохранит в зашифрованном виде с хэшированными путями
	
	// 3. Загрузка данных (асинхронно)
	
	ProfileType profile;  // ваш тип класса профиля
	bool loaded = await data.LoadA<ProfileType>(out profile, "data/player/profile");
	
	if (loaded)
		Console.WriteLine("Профиль успешно загружен!");
	else
		Console.WriteLine("Профиль не найден или повреждён");
	
	
	// 4. Отправка/получение по сети
	
	byte[] networkData = data.EncryptNetworkData(request);  // → готово к отправке
	
	// На стороне получателя:
	MemoryStream receivedStream = new MemoryStream();  // сюда приходят пакеты
	
	// Когда получили кусок данных:
	byte[] chunk = ...;  // полученный кусок
	receivedStream.Write(chunk, 0, chunk.Length);
	
	// Обрабатываем всё, что накопилось
	var remains = data.DecryptNetworkData<Request>(
		receivedStream.ToArray(),
		req =>
		{
			// Здесь ваша логика обработки запроса
			Console.WriteLine($"Получен запрос: {req.Key}");
		});
	
	// remains — это неполный кусок, который ещё не образовал целое сообщение
	receivedStream.SetLength(0);  // очищаем поток
	
	if (remains.Length > 0)
	{
		receivedStream.Write(remains.ToArray(), 0, remains.Length);
		// теперь при следующем пакете остаток будет учтён
	}
	
	
	// 5. Логирование действий
	
	data.Log += (logEvent) =>
	{
		if (logEvent.Level <= DataLogLevel.Info)
			Console.WriteLine(logEvent.Message);
	};
```

## 📖 Инициализация

### 🔧 Конструкторы

#### `PlayerData(string hash, ...)`
Создаёт экземпляр с строковым ключом шифрования.

```csharp
// Простейший вариант
var storage = new PlayerData("мой-секретный-ключ");

// Полная настройка
var storage = new PlayerData(
    hash: "мой-секретный-ключ",
    startPath: "C:/MyApp/Data",           // Куда сохранять файлы
    lengthNameDirectory: HexLength.Medium, // Длина хэша для папок (16 символов)
    lengthNameFile: HexLength.Short,      // Длина хэша для файлов (8 символов)
    offsetHashHex: -1                     // Автосмещение для уникальности хэшей
);
```

#### `PlayerData(byte[] hash, ...)`
То же самое, но с байтовым ключом (более безопасно).

```csharp
// Генерация ключа из байтов
var keyBytes = Encoding.UTF8.GetBytes("мой-секретный-ключ");
var storage = new PlayerData(keyBytes);
```

### ⚙️ Методы настройки (можно вызывать после создания)

#### `SetHash(string hash)` / `SetHash(byte[] hash)`
Устанавливает или изменяет ключ шифрования.

```csharp
var storage = new PlayerData("мой-временный-ключ");

// Позже меняем на постоянный (например, после ввода пароля пользователем)
storage.SetHash("мой-постоянный-ключ");

// Байтовый ключ (рекомендуется)
var secureHash = new byte[] { 0x01, 0x02, 0x03, ... };
storage.SetHash(secureHash);
```

> **⚠️ Важно:** При смене ключа старые данные не смогут быть расшифрованы!

#### `SetStartPath(string path)`
Задаёт корневую папку для хранения данных.

```csharp
// Для desktop-приложений
storage.SetStartPath(Path.Combine(
    Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
    "CompanyName",
	"YourAppName"
));

// Для мобильных приложений (Xamarin/MAUI)
storage.SetStartPath(FileSystem.AppDataDirectory);

// Проверка: путь будет создан, если не существует
```

#### `SetLengthNameDirectory(HexLength length)`  

#### `SetLengthNameFile(HexLength length)`
Настраивают длину хэшированных имён.

```csharp
// Возможные значения HexLength:
//Short  - 8 символов  (4 байта)
//Medium - 16 символов (8 байт)
//Long   - 32 символа  (16 байт)
//Full 	 - 64 символа  (32 байта) - по умолчанию

// Пример: компактные имена для мобильных устройств
storage.SetLengthNameDirectory(HexLength.Medium);
storage.SetLengthNameFile(HexLength.Medium);
// Путь "users/profile" → "a1b2e5f67890c3d4/e5f6a1b2c3d47890"
```

#### `SetOffsetHashHex(int offsetHex)`
Смещение в хэше для уменьшения коллизий.

```csharp
// Фиксированное смещение
storage.SetOffsetHashHex(10); // Начинать хэш с 10-го символа

// Автоматическое смещение (рекомендуется)
storage.SetOffsetHashHex(-1); // Смещение = длина хэшируемой строки

// Пример с offsetHashHex = -1:
// "users" (5 символов) → хэш начиная с 5-й позиции
// "profile" (7 символов) → хэш начиная с 7-й позиции
```

### 🎯 Примеры использования

#### Пример 1: Простое desktop-приложение
```csharp
public class AppStorage
{
    private readonly PlayerData storage;
    
    public AppStorage()
    {
        storage = new PlayerData(
            hash: "мой-секретный-ключ",
            startPath: Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
                "PixSTD",
                "MyApp"
            ),
            lengthNameDirectory: HexLength.Medium,
            lengthNameFile: HexLength.Medium,
            offsetHashHex: -1
        );
    }
}
```

#### Пример 2: Поэтапная настройка
```csharp
// Когда нужна гибкая настройка
var storage = new PlayerData("мой-секретный-ключ");

// Позже, после инициализации приложения
storage.SetHash("мой-секретный-ключ");
storage.SetStartPath(GetUserDataPath());
storage.SetLengthNameDirectory(HexLength.Medium);
storage.SetLengthNameFile(HexLength.Medium);
storage.SetOffsetHashHex(-1);
```

### ❗ Важные замечания

1. **Ключ шифрования**:
   - Байтовый ключ: Минимум 16 байт
   - Храните безопасно (не в коде!)
   - При утере ключа данные не восстановить

2. **Пути**:
   - Автоматически создаются недостающие папки
   - Можно использовать относительные пути

3. **Безопасность**:
   - Разные длины хэшей для папок и файлов усложняют анализ структуры
   - Смещение (`offsetHashHex`) предотвращает pattern analysis

## 💾 Сохранение и загрузка данных

### 📤 Сохранение объектов

#### `Save(object data, string encryptedPath, string unencryptedPath = "")`
Синхронное сохранение данных.

#### `SaveA(object data, string encryptedPath, string unencryptedPath = "")`
Асинхронное сохранение (рекомендуется).

```csharp
// Любой объект можно сохранить
var user = new { Name = "Иван", Age = 25, Email = "ivan@mail.com" };
var settings = new AppSettings { Theme = "Dark", Language = "RU" };

// Сохраняем в зашифрованном виде
storage.Save(user, "users/ivan/profile");
await storage.SaveA(settings, "app/settings");

// С сохранением в конкретную папку
await storage.SaveA(
    data: user,
    encryptedPath: "profile",           // хэшируется: "profile" → "e5f6a1b2c3d47890"
    unencryptedPath: "C:/backups/users" // Не хэшируется, используется как есть
);
```

### 📥 Загрузка объектов

#### `Load<T>(Action<T> apply, string encryptedPath, string unencryptedPath = "")`
Синхронная загрузка с обработкой через callback.

#### `LoadA<T>(Action<T> apply, string encryptedPath, string unencryptedPath = "")`
Асинхронная загрузка.

#### `Load<T>(out T value, string encryptedPath, string unencryptedPath = "")`
Загрузка с возвратом значения.

```csharp
// Способ 1: Через callback (рекомендуется для сложной логики)
storage.Load<AppSettings>(settings => 
{
    Console.WriteLine($"Тема: {settings.Theme}");
    ApplyTheme(settings.Theme);
}, "app/settings");

// Асинхронный вариант
await storage.LoadA<List<Document>>(documents =>
{
    foreach (var doc in documents)
        ProcessDocument(doc);
}, "projects/report/documents");

// Способ 2: Получение значения напрямую
if (storage.Load<User>(out var user, "users/ivan/profile"))
{
    Console.WriteLine($"Привет, {user.Name}!");
}
else
{
    Console.WriteLine("Пользователь не найден");
}

// Загрузка с указанием папки
storage.Load<BackupData>(
    data => RestoreBackup(data),
    encryptedPath: "backup",
    unencryptedPath: "D:/archives"
);
```

### 🔄 Работа с разными типами данных

```csharp
// Примитивные типы
await storage.SaveA(42, "config/max_items");
await storage.SaveA("Hello World", "messages/greeting");

// Коллекции
var scores = new Dictionary<string, int> 
{ 
    ["Иван"] = 100, 
    ["Мария"] = 95 
};
await storage.SaveA(scores, "game/scores");

// Сложные объекты
[MessagePackObject] // Использование атрибутов MessagePack уменьшает размер данных на 20-30%
public class Invoice
{
    [Key(0)]
    public string Number { get; set; }
    [Key(1)]
    public DateTime Date { get; set; }
    [Key(2)]
    public List<InvoiceItem> Items { get; set; }
    [Key(3)]
    public decimal Total { get; set; }
}

var invoice = new Invoice { /* ... */ };
await storage.SaveA(invoice, $"invoices/{invoice.Number}");
```

### 🎯 Пример использования
```csharp
public class UserService
{
    private readonly PlayerData storage;
    
    public async Task<User> GetUserAsync(string userId)
    {
        if (storage.Load<User>(out var user, $"users/{userId}/profile"))
            return user;
            
        // Если пользователя нет - создаём нового
        var newUser = new User { Id = userId, Created = DateTime.Now };
        await storage.SaveA(newUser, $"users/{userId}/profile");
        return newUser;
    }
    
    public async Task SaveUserSettingsAsync(string userId, UserSettings settings)
    {
        await storage.SaveA(settings, $"users/{userId}/settings");
    }
    
    public async Task<List<User>> GetAllUsersAsync()
    {
        var users = new List<User>();
        
        // Загружаем всех пользователей из папки users/
        var userFolders = storage.GetDirectories("users");
        foreach (var folder in userFolders)
        {
            if (storage.Load<User>(out var user, "profile", folder))
                users.Add(user);
        }
        
        return users;
    }
}
```

## 📁 Копирование файлов и папок

### 📄 Копирование файлов

#### `CopyFile(string fromEncryptedPath, string toEncryptedPath, ...)`
Копирует один файл в другое место.

```csharp
// Базовое копирование (перезаписывает существующий файл)
storage.CopyFile(
    fromEncryptedPath: "users/ivan/profile",
    toEncryptedPath: "backups/users/ivan_profile_backup"
);

// С указанием конкретных папок
storage.CopyFile(
    fromEncryptedPath: "report",
	toEncryptedPath: "archive/report_2026",
    fromUnencryptedPath: "C:/reports/2026",	// Исходная папка
    toUnencryptedPath: "D:/archive",       	// Конечная папка
    overwrite: true                        	// Перезаписать если существует
);

// Безопасное копирование (не перезаписывает)
storage.CopyFile(
    fromEncryptedPath: "config/settings",
    toEncryptedPath: "config/settings_backup",
    overwrite: false  // Если файл уже существует - копирование не выполнится
);
```

### 📂 Копирование папок (рекурсивное)

#### `CopyDirectory(string fromEncryptedPath, string toEncryptedPath, ...)`
Копирует все файлы из одной папки в другую.

```csharp
// Копирование всей папки
var (copied, total) = storage.CopyDirectory(
    fromEncryptedPath: "users",
    toEncryptedPath: "backups/2026/users"
);

Console.WriteLine($"Скопировано: {copied}/{total} файлов");

// Копирование с перезаписью
storage.CopyDirectory(
    fromEncryptedPath: "app/logs",
    toEncryptedPath: "archive/logs_january",
    overwrite: true  // Перезаписать существующие файлы
);

// Копирование между разными корневыми папками
storage.CopyDirectory(
    fromEncryptedPath: "data/project_x",
	toEncryptedPath: "backups/project_x",
    fromUnencryptedPath: "C:/projects/current",
    toUnencryptedPath: "D:/backups/archive"
);
```

### 🎯 Пример использования
```csharp
public class BackupService
{
    private readonly PlayerData storage;
    
    public void CreateUserBackup(string userId, string backupName)
    {
        var backupPath = $"backups/{DateTime.Now:yyyy-MM-dd}/{userId}";
        
        // Копируем все данные пользователя в бэкап
        storage.CopyDirectory(
            fromEncryptedPath: $"users/{userId}",
            toEncryptedPath: $"{backupPath}"
        );
        
        // Копируем отдельно важные файлы с метаданными
        var importantFiles = new[]
        {
            $"users/{userId}/profile",
            $"users/{userId}/settings",
            $"users/{userId}/subscription"
        };
        
        foreach (var file in importantFiles)
        {
			var fileName = PlayerData.GetNameFile(file);
			storage.CopyFile(
				fromEncryptedPath: file,
				toEncryptedPath: $"{backupPath}/critical/{fileName}"
			);
        }
    }
    
    public void RestoreUserBackup(string userId, string backupDate)
    {
        // Восстанавливаем из бэкапа
        storage.CopyDirectory(
            fromEncryptedPath: $"backups/{backupDate}/{userId}",
            toEncryptedPath: $"users/{userId}",
            overwrite: true  // Перезаписываем текущие данные
        );
    }
}
```

### 🔍 Поиск файлов и папок

#### `SearchFile(string encryptedPath, ...)` / `SearchDirectory(...)`
Проверяет существование файла или папки.

```csharp
// Проверка существования файла
if (storage.SearchFile("users/ivan/profile"))
{
    Console.WriteLine("Профиль пользователя существует");
}

// Проверка существования папки
if (storage.SearchDirectory("users"))
{
    var userCount = storage.QuantityDirectories("users");
    Console.WriteLine($"Найдено пользователей: {userCount}");
}

// Поиск с указанием конкретной папки
var exists = storage.SearchFile(
    encryptedPath: "report",
    unencryptedPath: "C:/documents/reports"
);
```

#### `GetFiles(string encryptedPath, ...)` / `GetDirectories(...)`
Получает список файлов или папок.

```csharp
// Получить все файлы в папке
var allFiles = storage.GetFiles("users/ivan");
foreach (var file in allFiles)
{
    Console.WriteLine($"Файл: {file}");
}

// Получить все подпапки
var userFolders = storage.GetDirectories("users");
foreach (var folder in userFolders)
{
    var userName = PlayerData.GetNameDirectory(folder);
    Console.WriteLine($"Папка пользователя: {userName}");
}

// Получить только определённые файлы
var imageFiles = storage.GetFiles("gallery")
    .Where(f => f.EndsWith(".jpg") || f.EndsWith(".png"));
```

#### `QuantityFiles(...)` / `QuantityDirectories(...)`
Подсчитывает количество файлов или папок.

```csharp
// Подсчёт файлов в проекте
var fileCount = storage.QuantityFiles("projects/report");
var folderCount = storage.QuantityDirectories("projects");

Console.WriteLine($"Проект содержит: {fileCount} файлов в {folderCount} папках");

// Мониторинг использования хранилища
public StorageInfo GetStorageInfo()
{
    return new StorageInfo
    {
        TotalFiles = storage.QuantityFiles(""),
        TotalDirectories = storage.QuantityDirectories(""),
        UserFiles = storage.QuantityFiles("users"),
        BackupFiles = storage.QuantityFiles("backups")
    };
}
```

### 🗑️ Удаление файлов и папок

#### `DeleteFile(string encryptedPath, ...)` / `DeleteDirectory(...)`
Удаляет файлы и папки (рекурсивно).

```csharp
// Удаление одного файла
storage.DeleteFile("temp/cache_data");

// Удаление папки со всем содержимым
storage.DeleteDirectory("old_backups");

// Очистка временных файлов
public void CleanupTempFiles()
{
    var tempFiles = storage.GetFiles("temp");
    foreach (var file in tempFiles)
    {
        // Удаляем только старые файлы
        if (IsFileOlderThan(file, TimeSpan.FromDays(7)))
        {
            storage.DeleteFile("", file);
        }
    }
}
```

## 🛠️ Вспомогательные функции

### 📂 Работа с путями

#### `NormalizePath(string path)`
Нормализует путь, заменяя обратные слеши и убирая двойные слеши.

```csharp
// Примеры нормализации
var path1 = PlayerData.NormalizePath("C:\\Users\\ivan\\data");
// → "C:/Users/ivan/data"

var path2 = PlayerData.NormalizePath("folder//subfolder\\\\file.txt");
// → "folder/subfolder/file.txt"

var path3 = PlayerData.NormalizePath(@"app\data\");
// → "app/data/"
```

#### `GetNameFile(string path)`
Извлекает имя файла из пути.

```csharp
// Получение имени файла
var fileName = PlayerData.GetNameFile("users/ivan/profile.dat");
// → "profile.dat"

var fileName2 = PlayerData.GetNameFile("C:/projects/report.pdf");
// → "report.pdf"

var fileName3 = PlayerData.GetNameFile("folder/subfolder/");
// → "" (пустая строка для папок)
```

#### `GetNameDirectory(string path)`
Извлекает имя последней папки из пути.

```csharp
// Получение имени папки
var dirName = PlayerData.GetNameDirectory("users/ivan/profile.dat");
// → "ivan"

var dirName2 = PlayerData.GetNameDirectory("projects/2024/january/");
// → "january"

var dirName3 = PlayerData.GetNameDirectory("file.txt");
// → "" (пустая строка для файлов в корне)
```

#### `GetPathToDirectory(string path)`
Получает путь к родительской папке.

```csharp
// Получение пути к родительской папке
var parentPath = PlayerData.GetPathToDirectory("users/ivan/profile.dat");
// → "users/ivan/"

var parentPath2 = PlayerData.GetPathToDirectory("projects/");
// → "projects/"

var parentPath3 = PlayerData.GetPathToDirectory("file.txt");
// → "" (корневая папка)
```

#### `GetCutPath(string path, int keepRight = 1)`
Оставляет только указанное количество сегментов пути справа.

```csharp
// Сокращение пути
var shortPath = PlayerData.GetCutPath("projects/2024/january/report.pdf", 2);
// → "january/report.pdf"

var shortPath2 = PlayerData.GetCutPath("users/ivan/documents/contract.pdf", 1);
// → "contract.pdf"

var shortPath3 = PlayerData.GetCutPath("folder1/folder2/folder3/", 3);
// → "folder1/folder2/folder3/"
```

### 🔄 Конвертация объектов

#### `Convert(object data)` → `byte[]`
Сериализует любой объект в байты с помощью MessagePack.

```csharp
// Сериализация объектов
var user = new { Name = "Иван", Age = 30 };
byte[] userBytes = PlayerData.Convert(user);

var settings = new AppSettings { Theme = "Dark" };
byte[] settingsBytes = PlayerData.Convert(settings);

// Работа с коллекциями
var numbers = new List<int> { 1, 2, 3, 4, 5 };
byte[] numbersBytes = PlayerData.Convert(numbers);

// Словари тоже поддерживаются
var dict = new Dictionary<string, string>
{
    ["key1"] = "value1",
    ["key2"] = "value2"
};
byte[] dictBytes = PlayerData.Convert(dict);

// Оптимизация с атрибутами MessagePack
[MessagePackObject]
public class OptimizedData
{
    [Key(0)] public int Id { get; set; }
    [Key(1)] public string Name { get; set; }
    [Key(2)] public DateTime Created { get; set; }
}

var optimized = new OptimizedData { Id = 1, Name = "Test" };
byte[] optimizedBytes = PlayerData.Convert(optimized); // Более компактный формат
```

#### `Convert<T>(byte[] data)` → `T`
Десериализует байты обратно в объект указанного типа.

```csharp
// Десериализация
byte[] userData = // ... полученные байты
var user = PlayerData.Convert<User>(userData);

// Работа с коллекциями
byte[] listData = // ... байты списка
var numbers = PlayerData.Convert<List<int>>(listData);
```

#### `Convert(byte[] data, Type type)` → `object`
Динамическая десериализация по типу.

```csharp
// Динамическая десериализация
Type targetType = typeof(UserProfile);
byte[] data = // ... полученные байты

object result = PlayerData.Convert(data, targetType);
if (result is UserProfile profile)
{
    Console.WriteLine($"Пользователь: {profile.Name}");
}
```

## 🔐 Шифрование и работа с сетью

### 🔒 Локальное шифрование

#### `Encrypt(byte[] plaintext)` → `byte[]`
Шифрует массив байтов

```csharp
// Шифрование
var user = new { Name = "Иван", Password = "secret123" };
byte[] serialized = PlayerData.Convert(user);
byte[] encryptedUser = storage.Encrypt(serialized);
```

#### `Decrypt(byte[] data)` → `byte[]`
Расшифровывает массив байтов, зашифрованный методом `Encrypt`.

```csharp
// Расшифровка и десериализация
byte[] encryptedUserData = // ... зашифрованные данные пользователя
byte[] decryptedData = storage.Decrypt(encryptedUserData);
var user = PlayerData.Convert<User>(decryptedData);
```

### 🌐 Сетевое взаимодействие

#### `EncryptNetworkData(object data)` → `byte[]`
Подготавливает объект для сетевой передачи: сериализует, шифрует и добавляет маркеры.

```csharp
// Подготовка данных для отправки по сети
var message = new ChatMessage
{
    Sender = "Иван",
    Text = "Привет!",
    Timestamp = DateTime.UtcNow
};

byte[] networkPacket = storage.EncryptNetworkData(message);
// → маркер_начало + зашифрованные_данные + маркер_конец

// Отправка через сокет
public async Task SendMessageAsync(NetworkStream stream, object data)
{
    byte[] packet = storage.EncryptNetworkData(data);
    await stream.WriteAsync(packet, 0, packet.Length);
    
    Console.WriteLine($"Отправлено пакетов: {packet.Length} байт");
}
```

#### `DecryptNetworkData<T>(ReadOnlyMemory<byte> data, Action<T> action)`
Обрабатывает входящие сетевые данные, автоматически выделяя целые сообщения.

```csharp
// Обработка входящих сетевых данных
public async Task ReceiveDataAsync(NetworkStream stream)
{
    var buffer = new byte[8192];
    var memoryStream = new MemoryStream();
    
    while (true)
    {
        int bytesRead = await stream.ReadAsync(buffer, 0, buffer.Length);
        if (bytesRead == 0) break;
        
        memoryStream.Write(buffer, 0, bytesRead);
        
        // Обрабатываем все полные сообщения
        var remaining = storage.DecryptNetworkData<ChatMessage>(memoryStream.ToArray(), message =>
        {
            Console.WriteLine($"[{message.Sender}]: {message.Text}");
            ProcessMessage(message);
        });
        
        // Сохраняем неполное сообщение для следующего пакета
		
		memoryStream.SetLength(0);
        if (remaining.Length > 0)
			memoryStream.Write(remaining.ToArray(), 0, remaining.Length);
    }
}
```

# 📝 Логирование и обработка событий

## 🎯 Система логирования

### 📊 Уровни логирования (DataLogLevel)

```csharp
// Все доступные уровни логирования
public enum DataLogLevel
{
    Error = 0,     // Критические ошибки
    Warning = 1,   // Предупреждения
    Info = 2,      // Информационные сообщения
    Debug = 3,     // Отладочная информация
    Trace = 4      // Детальная трассировка
}

// Пример использования уровней
public void ConfigureLogging(DataLogLevel minLevel)
{
    // Фильтрация по уровню
    storage.Log += (logEvent) => 
    {
        if (logEvent.Level <= minLevel)
        {
            Console.WriteLine($"[{logEvent.Level}] {logEvent.Message}");
        }
    };
}
```

### 🔔 Событие Log

#### Подписка на события логирования
```csharp
// Базовая подписка
var storage = new PlayerData("мой-секретный-ключ");
storage.Log += OnLogEvent;

// Обработчик событий
private void OnLogEvent(PlayerDataLogEvent logEvent)
{
    Console.WriteLine($"[{logEvent.Timestamp:HH:mm:ss}] {logEvent.Level}: {logEvent.Message}");
    
    if (logEvent.Exception != null)
    {
        Console.WriteLine($"Исключение: {logEvent.Exception.Message}");
        Console.WriteLine($"Стек: {logEvent.Exception.StackTrace}");
    }
}

// Несколько обработчиков
storage.Log += LogToConsole;
storage.Log += LogToFile;

// Отписка от события
storage.Log -= LogToConsole;
```