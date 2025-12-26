# PixSTD.DataUtilities
**Официальная библиотека от PixSTD**

📦 NuGet: `PixSTD.DataUtilities`  
👨💻 Автор: PixSTD  
🐙 Исходный код: https://github.com/PixSTD/DataUtilities.dll  
📄 Лицензия: MIT

Библиотека для работы с данными.

## 🚀 Быстрый старт
	using DataUtilities;

	// 1. Создайте экземпляр
	var data = new PlayerData();

	// 2. Инициализация
	data.Initialize("your-secret-key");

	// 3. Сохранение
	await data.SaveA(gameState, "data/player/profile");
	
	// 4. Загрузка
	await data.LoadA(out string profile, "data/player/profile");

## Инициализация
	void Initialize(string hash)
	void Initialize(byte[] hash)
	
## Сохранение данных
	void Save(object data, string encryptedPath, string unencryptedPath = "")
	Task SaveA(object data, string encryptedPath, string unencryptedPath = "")

## Загрузка данных
	bool Load<T>(Action<T> apply, string encryptedPath, string unencryptedPath = "")
	Task<bool> LoadA<T>(Action<T> apply, string encryptedPath, string unencryptedPath = "")
	bool LoadA<T>(out T value, string encryptedPath, string unencryptedPath = "")

## Работа с файлами
	int QuantityFiles(string encryptedPath, string unencryptedPath = "")
	string[] GetFiles(string encryptedPath, string unencryptedPath = "")
	bool SearchFile(string encryptedPath, string unencryptedPath = "")
	void DeleteFile(string encryptedPath, string unencryptedPath = "")

## Работа с директориями
	int QuantityDirectories(string encryptedPath, string unencryptedPath = "")
	string[] GetDirectories(string encryptedPath, string unencryptedPath = "")
	bool SearchDirectory(string encryptedPath, string unencryptedPath = "")
	void DeleteDirectory(string encryptedPath, string unencryptedPath = "")

## Утилиты путей
	string GetRightPath(string path)
	string GetNameFile(string path)
	string GetNameDirectory(string path)
	string GetPathToDirectory(string path)
	string GetCutPath(string path, int keepRight = 1)

## Конвертация
	byte[] Convert(object data)
	T Convert<T>(byte[] data)
	object Convert(byte[] data, Type type)

## Шифрование
	byte[] Encrypt(byte[] plaintext)
	byte[] Decrypt(byte[] data)