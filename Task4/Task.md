а) Створити абстрактний базовий клас Triad з віртуальними функціями
збільшення на 1 і друку значення. Створити похідні класи Date (поточна
дата), Timе (поточний час) з власною реалізацією вищенаведених функцій.
b) Створити клас Memories (набір), що містить масив пар (дата-час)
об’єктів цих класів в динамічній пам'яті. Передбачити можливість виводу
всіх елементів масиву і визначення першої та останньої подій. Додаткове
завдання: доповнити клас методами сортування за деяким критерієм, виводу
у файл та зчитування з файлу.

Код: 
```
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

abstract class Triad
{
    public abstract void Increase();
    public abstract void Print();
    public abstract string ToString();
}

class DateTriad : Triad
{
    public int Day, Month, Year;
    public DateTriad(int d, int m, int y)
    {
        Day = d; Month = m; Year = y;
    }

    public override void Increase()
    {
        Day++;
        if (Day > 31) { Day = 1; Month++; }
        if (Month > 12) { Month = 1; Year++; }
    }

    public override void Print()
    {
        Console.WriteLine(ToString());
    }

    public override string ToString()
    {
        return $"{Day:00}.{Month:00}.{Year}";
    }
}

class TimeTriad : Triad
{
    public int Hour, Minute, Second;
    public TimeTriad(int h, int m, int s)
    {
        Hour = h; Minute = m; Second = s;
    }

    public override void Increase()
    {
        Second++;
        if (Second >= 60) { Second = 0; Minute++; }
        if (Minute >= 60) { Minute = 0; Hour++; }
        if (Hour >= 24) { Hour = 0; }
    }

    public override void Print()
    {
        Console.WriteLine(ToString());
    }

    public override string ToString()
    {
        return $"{Hour:00}:{Minute:00}:{Second:00}";
    }
}

class MemoryEntry
{
    public DateTriad Date;
    public TimeTriad Time;

    public MemoryEntry(DateTriad d, TimeTriad t)
    {
        Date = d;
        Time = t;
    }

    public void Print()
    {
        Console.WriteLine($"{Date.ToString()} {Time.ToString()}");
    }
}

class Memories
{
    private List<MemoryEntry> events = new List<MemoryEntry>();

    public void Add(MemoryEntry entry)
    {
        events.Add(entry);
    }

    public void PrintAll()
    {
        Console.WriteLine("\n🔹 Усі події:");
        foreach (var ev in events)
            ev.Print();
    }

    public void PrintFirstLast()
    {
        if (events.Count == 0) return;
        Console.WriteLine("\n🔸 Перша подія:");
        events.First().Print();
        Console.WriteLine("🔸 Остання подія:");
        events.Last().Print();
    }

    public void SortByDate()
    {
        events.Sort((a, b) => a.Date.ToString().CompareTo(b.Date.ToString()));
    }

    public void SaveToFile(string path)
    {
        using (var writer = new StreamWriter(path))
        {
            foreach (var ev in events)
                writer.WriteLine($"{ev.Date.ToString()} {ev.Time.ToString()}");
        }
    }

    public void LoadFromFile(string path)
    {
        events.Clear();
        foreach (var line in File.ReadAllLines(path))
        {
            var parts = line.Split(' ');
            var dateParts = parts[0].Split('.');
            var timeParts = parts[1].Split(':');

            Add(new MemoryEntry(
                new DateTriad(int.Parse(dateParts[0]), int.Parse(dateParts[1]), int.Parse(dateParts[2])),
                new TimeTriad(int.Parse(timeParts[0]), int.Parse(timeParts[1]), int.Parse(timeParts[2]))
            ));
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("=== Завдання А ===");

        DateTriad date = new DateTriad(31, 12, 2023);
        TimeTriad time = new TimeTriad(23, 59, 59);

        Console.Write("Початкова дата: ");
        date.Print();
        date.Increase();
        Console.Write("Після збільшення: ");
        date.Print();

        Console.Write("Початковий час: ");
        time.Print();
        time.Increase();
        Console.Write("Після збільшення: ");
        time.Print();

        Console.WriteLine("\n\n=== Завдання B ===");
        Memories mem = new Memories();

        mem.Add(new MemoryEntry(new DateTriad(1, 1, 2024), new TimeTriad(12, 0, 0)));
        mem.Add(new MemoryEntry(new DateTriad(31, 12, 2023), new TimeTriad(23, 59, 59)));
        mem.Add(new MemoryEntry(new DateTriad(15, 5, 2024), new TimeTriad(8, 30, 0)));

        mem.PrintAll();
        mem.PrintFirstLast();

        Console.WriteLine("\n🔃 Після сортування:");
        mem.SortByDate();
        mem.PrintAll();

        mem.SaveToFile("events.txt");
        Console.WriteLine("\n💾 Події збережено у файл 'events.txt'");
    }
}

```

Результат: 

![image](https://github.com/user-attachments/assets/3b62d797-570f-4039-93ed-7f954d619042)
