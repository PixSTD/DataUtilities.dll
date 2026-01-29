# Changelog

Все заметные изменения в этом проекте документируются в этом файле.

## [0.1.4] - 2026-01-30

### Добавлено (Added)
- **Новый enum `HexLength`** со значениями:
  - `Short` = 8 символов (4 байта)
  - `Medium` = 16 символов (8 байт)
  - `Long` = 32 символа (16 байт) 
  - `Full` = 64 символа (32 байта) *(по умолчанию)*

- **Обновлённые конструкторы `PlayerData`** с поддержкой `HexLength`:
  ```csharp
  PlayerData(string hash, string startPath = "", 
			 HexLength lengthNameDirectory = HexLength.Long,
             HexLength lengthNameFile = HexLength.Full,
             int offsetHashHex = 0)```

- Перегрузки метода HashHex теперь работают с HexLength вместо int lengthByte:
  ```csharp
  public static string HashHex(string data, HexLength length = HexLength.Full, int offsetChars = 0) //
  public static string HashHex(byte[] data, HexLength length = HexLength.Full, int offsetChars = 0) //теперь работает с символами hex, а не с байтами```

### Исправлено (Fixed)
- Исправлены недочеты из за которых могли быть ошибки