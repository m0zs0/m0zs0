# Whack-a-Mole WPF Játék

A játékban egy 3x3-as mátrixban lyukak vannak, amelyekből véletlenszerűen állatok bukkannak fel. 

A cél, hogy az állatokra kattintsunk pontokért (+1). 

A játék 10 pont elérésekor ér véget. 

A nehézségi szint egy csúszka (slider) segítségével állítható, ahol a könnyebb szinteken több állat jelenik meg egyszerre, a nehezebbeken pedig kevesebb és rövidebb ideig.

## A projekt felépítése

**A felhasználói felületet az XAML definiálja, amely egy rácsot (Grid) tartalmaz három sorral:**

- Első sor: Pontszám és nehézség beállítása.
- Második sor: 3x3-as játékmező (UniformGrid).
- Harmadik sor: "Új játék" gomb.

A csúszka (Slider) 4 fokozattal rendelkezik: "Könnyű", "Normál", "Nehéz", "Profi".

**MainWindow.xaml.cs**
A logika aszinkron módon (async/await) van implementálva, hogy az állatok megjelenése és eltűnése függetlenül, valós idejű hatást keltve történjen.

Főbb jellemzők és működés

A InitializeGame metódus létrehozza a 3x3-as gombmátrixot:

```cs
private void InitializeGame()
{
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            Button btn = new Button
            {
                Content = new Image { Source = new BitmapImage(new Uri(holeImage)) },
                Background = Brushes.White,
                Tag = "hole"
            };
            btn.Click += Button_Click;
            gameButtons[i, j] = btn;
            GameGrid.Children.Add(btn);
        }
    }
}
```

`Gombok`: Minden gomb egy lyuk képet (hole.png) tartalmaz kezdetben, és a Tag tulajdonsága jelzi az állapotát ("hole" vagy "animal").
`Eseménykezelő`: A Button_Click kezeli a kattintásokat.

**Nehézségi szint kezelése**
A DifficultySlider_ValueChanged metódus frissíti a nehézségi szint szövegét:

```cs
private void DifficultySlider_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
{
    if (DifficultyText != null)
    {
        string[] difficultyLevels = { "Könnyű", "Normál", "Nehéz", "Profi" };
        int level = (int)DifficultySlider.Value;
        DifficultyText.Text = difficultyLevels[level];
        System.Diagnostics.Debug.WriteLine($"Difficulty level: {DifficultySlider.Value}");
    }
}
```
`Slider`: 0-tól 3-ig terjedő értékek ("Könnyű" = 0, "Profi" = 3).

Null-ellenőrzés: Biztosítja, hogy a DifficultyText létezik, elkerülve a NullReferenceException-t.
A `System.Diagnostics.Debug.WriteLine($"Difficulty level: {DifficultySlider.Value}");` utasítással az Output ablakba tudunk mindenféle futásközbeni adatot kiírni. 

**Aszinkron működés részletes magyarázata**

Az async/await lehetővé teszi, hogy időzített műveleteket (pl. állatok megjelenése, eltűnése) végezzünk anélkül, hogy blokkolnánk az UI szálat. Ez kulcsfontosságú a WPF-ben, mert az UI manipuláció csak az UI szálon történhet.

`StartGame` - A játék indítása
Ez a metódus indítja el a játékot:

```cs
private async void StartGame()
{
    isGameRunning = true;
    score = 0;
    ScoreText.Text = "0";

    List<Task> positionTasks = new List<Task>();
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            positionTasks.Add(RunPositionAsync(i, j));
        }
    }

    while (isGameRunning && score < 10)
    {
        await Task.Delay(100);
    }

    isGameRunning = false;
    ResetAnimals();

    if (score >= 10)
    {
        MessageBox.Show("Gratulálok! Nyertél!");
    }
}
```

`async void`: Ez egy aszinkron metódus, amely nem tér vissza értékkel, és az UI szálon fut.
`List<Task>`: Minden pozícióhoz (3x3 = 9) egy külön Task-ot indítunk a RunPositionAsync metódussal, így az állatok függetlenül működnek.
`await Task.Delay(100)`: Egy végtelen ciklusban 100 ms-onként ellenőrzi, hogy a játék fut-e és a pontszám elérte-e a 10-et. Az await itt szünetelteti a ciklust anélkül, hogy blokkolná az UI-t.
Az `isGameRunning = false` leállítja az összes futó feladatot.

`RunPositionAsync` - Pozíciónkénti logika
Ez a metódus kezeli az állatok felbukkanását egy adott pozícióban:

```cs
private async Task RunPositionAsync(int row, int col)
{
    while (isGameRunning)
    {
        int maxAnimals = 4 - (int)DifficultySlider.Value; // 4, 3, 2, 1
        //int currentAnimals = gameButtons.Cast<Button>().Count(b => b.Tag.ToString() == "animal");
        int currentAnimals = 0;
        foreach (Button b in gameButtons)
        {
            if (b.Tag.ToString() == "animal")
            {
                currentAnimals++;
            }
        }

        if (currentAnimals < maxAnimals)
        {
            double difficulty = DifficultySlider.Value;
            double waitTime = random.NextDouble() * (1 + difficulty) + 0.5; // Könnyű: 0.5-1.5s, Profi: 0.5-3.5s
            await Task.Delay(TimeSpan.FromSeconds(waitTime));

            if (isGameRunning)
            {
                await ShowAnimalAsync(row, col);
            }
        }
        else
        {
            await Task.Delay(100);
        }
    }
}
```

`async Task`: Ez egy aszinkron metódus, amely Task-ot ad vissza, lehetővé téve a párhuzamos futást.
`while (isGameRunning)`: Addig fut, amíg a játék tart.
`maxAnimals`: A nehézségtől függően korlátozza az egyszerre látható állatok számát (Könnyű: 4, Profi: 1).
`currentAnimals`: Megszámolja az aktuálisan látható állatokat a gameButtons mátrixból.
`await Task.Delay(TimeSpan.FromSeconds(waitTime))`: Véletlenszerű időt vár (pl. Profi szinten 0.5-3.5s), mielőtt megpróbálna állatot megjeleníteni. Ez biztosítja, hogy ne jelenjenek meg túl gyakran állatok a nehezebb szinteken.
`await ShowAnimalAsync`: Ha van hely (currentAnimals < maxAnimals), meghívja az állat megjelenítését.

`ShowAnimalAsync` - Állat megjelenítése és eltűnése
Ez a metódus kezeli egy állat megjelenését és eltűnését:

```cs
private async Task ShowAnimalAsync(int row, int col)
{
    Button btn = gameButtons[row, col];
    if (btn.Tag.ToString() == "hole")
    {
        double difficulty = DifficultySlider.Value;
        double showTime = random.NextDouble() * (2.5 - difficulty * 0.75) + 0.25; // Könnyű: 2.5s, Profi: 0.25s

        btn.Content = new Image
        {
            Source = new BitmapImage(new Uri(animalImage)),
            Width = 90,
            Height = 90,
            Stretch = Stretch.Uniform
        };
        btn.Tag = "animal";

        await Task.Delay(TimeSpan.FromSeconds(showTime));

        if (btn.Tag.ToString() == "animal")
        {
            btn.Content = new Image
            {
                Source = new BitmapImage(new Uri(holeImage)),
                Width = 90,
                Height = 90,
                Stretch = Stretch.Uniform
            };
            btn.Tag = "hole";
        }
    }
}
```
`if (btn.Tag.ToString() == "hole")`: Csak üres lyukon jelenhet meg állat.
`showTime`: Véletlenszerű időtartam, amivel biztosítjuk, hogy a "Profi" szinten nagyon rövid ideig (0.25-0.5s) látható az állat.
`await Task.Delay(TimeSpan.FromSeconds(showTime))`: Ez a sor szünetelteti a metódust a megadott ideig, miközben az UI szál szabad marad. Az await itt azt jelenti, hogy a metódus "várakozik", de nem blokkol, és az idő letelte után folytatja a végrehajtást.
`Visszaállítás`: Ha az állatot nem kattintották le, visszaáll lyukká.

Miért fontos az await?

Az await kulcsszó lehetővé teszi, hogy a kód "várjon" egy aszinkron művelet (pl. Task.Delay) befejezésére anélkül, hogy az UI szál blokkolódna. Ez biztosítja, hogy a játék közben a felület reagáljon a kattintásokra, és az állatok függetlenül mozogjanak.

`Kattintás kezelése`
A Button_Click metódus kezeli a pontszerzést:

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
    if (!isGameRunning) return;

    Button btn = (Button)sender;
    if (btn.Tag.ToString() == "animal")
    {
        score++;
        btn.Content = new Image { Source = new BitmapImage(new Uri(holeImage)) };
        btn.Tag = "hole";
    }

    ScoreText.Text = score.ToString();
} 
```
`Logika`: +1 pont állatra kattintásért.

Teljes XAML kód

```cs
<Window x:Class="Wpf_1_WhacAMole.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Wpf_1_WhacAMole"
        mc:Ignorable="d"
        Title="Whack-a-Mole" Height="450" Width="450">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- Pontszám és nehézség -->
        <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="10">
            <TextBlock Text="Pontszám: " FontSize="16"/>
            <TextBlock x:Name="ScoreText" Text="0" FontSize="16"/>
            <TextBlock Text="Nehézség:" Margin="20,0,0,0"/>
            <Slider x:Name="DifficultySlider" Minimum="0" Maximum="3" Value="1" Width="100" Margin="10,0" 
                    SmallChange="1" LargeChange="1" TickFrequency="1" IsSnapToTickEnabled="True" 
                    ValueChanged="DifficultySlider_ValueChanged"/>
            <TextBlock x:Name="DifficultyText" Text="Normál" FontSize="16" Margin="10,0"/>
        </StackPanel>

        <!-- Játékmező -->
        <UniformGrid x:Name="GameGrid" Grid.Row="1" Rows="3" Columns="3" Margin="10" Width="300" Height="300">
            <!-- A gombok code-behind-ban lesznek létrehozva -->
        </UniformGrid>

        <!-- Újraindítás gomb -->
        <Button Grid.Row="2" Content="Új játék" Click="RestartGame" Margin="10"/>
    </Grid>
</Window>
```
```cs
using System.Text;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using System.Windows.Threading;

namespace Wpf_1_WhacAMole
{
    public partial class MainWindow : Window
    {
        private Button[,] gameButtons = new Button[3, 3];
        private Random random = new Random();
        private int score = 0;
        private string holeImage = "pack://application:,,,/Images/hole.png";
        private string animalImage = "pack://application:,,,/Images/animal.png";
        private bool isGameRunning = false;

        public MainWindow()
        {
            InitializeComponent();
            InitializeGame();
        }

        private void InitializeGame()
        {
            // Gombok létrehozása
            for (int i = 0; i < 3; i++)
            {
                for (int j = 0; j < 3; j++)
                {
                    Button btn = new Button
                    {
                        Content = new Image { Source = new BitmapImage(new Uri(holeImage)) },
                        Background = Brushes.White,
                        Tag = "hole"
                    };
                    btn.Click += Button_Click;
                    gameButtons[i, j] = btn;
                    GameGrid.Children.Add(btn);
                }
            }

            
        }

        private void DifficultySlider_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
        {
            // Ellenőrizzük, hogy a DifficultyText létezik-e
            if (DifficultyText != null)
            {
                string[] difficultyLevels = { "Könnyű", "Normál", "Nehéz", "Profi" };
                int level = (int)DifficultySlider.Value;
                DifficultyText.Text = difficultyLevels[level];
                System.Diagnostics.Debug.WriteLine($"Difficulty level: {DifficultySlider.Value}");
            }
        }


        private async Task ShowAnimalAsync(int row, int col)
        {
            Button btn = gameButtons[row, col];
            if (btn.Tag.ToString() == "hole")
            {
                double difficulty = DifficultySlider.Value; // 0-3
                // Megjelenési idő
                double showTime = random.NextDouble() * (2.5 - difficulty * 0.75) + 0.25; // Könnyű: 2.5s, Profi: 0.25s

                btn.Content = new Image
                {
                    Source = new BitmapImage(new Uri(animalImage)),
                    Width = 90,
                    Height = 90,
                    Stretch = Stretch.Uniform
                };
                btn.Tag = "animal";

                await Task.Delay(TimeSpan.FromSeconds(showTime));

                if (btn.Tag.ToString() == "animal")
                {
                    btn.Content = new Image
                    {
                        Source = new BitmapImage(new Uri(holeImage)),
                        Width = 90,
                        Height = 90,
                        Stretch = Stretch.Uniform
                    };
                    btn.Tag = "hole";
                }
            }
        }

        private async void StartGame()
        {
            isGameRunning = true;
            score = 0;
            ScoreText.Text = "0";

            // Minden pozícióhoz külön feladat indítása
            List<Task> positionTasks = new List<Task>();
            for (int i = 0; i < 3; i++)
            {
                for (int j = 0; j < 3; j++)
                {
                    positionTasks.Add(RunPositionAsync(i, j));
                }
            }

            // Várjuk, amíg a játék véget ér (10 pont vagy kézi leállítás)
            while (isGameRunning && score < 10)
            {
                await Task.Delay(100); // Kis szünet, hogy ne terhelje a CPU-t
            }

            isGameRunning = false; // Leállítja az összes pozíció futását
            ResetAnimals();

            if (score >= 10)
            {
                MessageBox.Show("Gratulálok! Nyertél!");
            }
        }

        private async Task RunPositionAsync(int row, int col)
        {
            while (isGameRunning)
            {
                // Maximális állatok száma a nehézség alapján
                int maxAnimals = 4 - (int)DifficultySlider.Value; // 4, 3, 2, 1

                //int currentAnimals = gameButtons.Cast<Button>().Count(b => b.Tag.ToString() == "animal");
                int currentAnimals = 0;
                foreach (Button b in gameButtons)
                {
                    if (b.Tag.ToString() == "animal")
                    {
                        currentAnimals++;
                    }
                }

                if (currentAnimals < maxAnimals) // Csak akkor jelenik meg új állat, ha van hely
                {
                    // Nehézség alapján várakozási idő: Profi szinten hosszabb szünet
                    double difficulty = DifficultySlider.Value;
                    double waitTime = random.NextDouble() * (1 + difficulty) + 0.5; // Könnyű: 0.5-1.5s, Profi: 0.5-3.5s
                    await Task.Delay(TimeSpan.FromSeconds(waitTime));

                    if (isGameRunning)
                    {
                        await ShowAnimalAsync(row, col);
                    }
                }
                else
                {
                    await Task.Delay(100); // Kis várakozás, ha nincs hely
                }
            }
        }

        private void Button_Click(object sender, RoutedEventArgs e)
        {
            if (!isGameRunning) return;

            Button btn = (Button)sender;
            if (btn.Tag.ToString() == "animal")
            {
                score++;
                btn.Content = new Image { Source = new BitmapImage(new Uri(holeImage)) };
                btn.Tag = "hole";
            }
            
            ScoreText.Text = score.ToString();
        }

        private void RestartGame(object sender, RoutedEventArgs e)
        {
            isGameRunning = false; // Megállítja a futó ciklusokat
            StartGame();
        }

        private void ResetAnimals()
        {
            foreach (Button btn in gameButtons)
            {
                if (btn.Tag.ToString() == "animal")
                {
                    btn.Content = new Image { Source = new BitmapImage(new Uri(holeImage)) };
                    btn.Tag = "hole";
                }
            }
        }
    }
}
```
