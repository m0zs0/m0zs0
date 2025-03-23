# Whac A Mole Sync - rossz megoldás
**A képek projekthez adása**
Képek hozzáadása a projekt mappájához:
- Solution Explorer-ben Projektnevének gyorsmenüje / "Add" -> "New Folder": Images
- Images gyorsmenüje / "Add" -> "Existing Item": Válaszd ki a hozzáadni kívánt képeket.
- A hozzáadott képre gyorsmenüje / "Properties" / "Build Action": "Resource"
- A képet ezután a következő URI formátummal érheted el: "pack://application:,,,/Images/hole.png"
ahol:
    - pack://application: Ez a prefix jelzi, hogy az erőforrás az alkalmazás csomagjában található.
    - ,,, A három vessző elválasztó jelzi, hogy az erőforrás az alkalmazás gyökérkönyvtárában található.
    - /Images/hole.png Ez az útvonal az erőforrás helyét jelzi az alkalmazás csomagjában.


## XAML Felület
### Felépítés: A főablak egy Grid-et tartalmaz, amely 3 részből áll:
- Felső panel: Pontszám és nehézségszint beállító (Slider).
- Játékmező: 3x3-as rács (UniformGrid), ahol a gombok dinamikusan jönnek létre.
- Újraindítás gomb: A játékot újrakezdő gomb.

```console
<Window x:Class="Wpf_1_WhacAMole_sync.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Wpf_1_WhacAMole_sync"
        mc:Ignorable="d"
        Title="Whack-a-Mole sync" Height="450" Width="450">
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

## Metódusok és feladatuk (C# Code-Behind)
### 1. InitializeGame()

Feladat: Létrehozza a 3x3-as játékmezőt.
```cs
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
```

Minden gombhoz hozzárendel egy "lyuk" képet (holeImage).

A gombok Tag tulajdonságát "hole"-ra állítja (jelzi, hogy nincs állat a lyukban).

Hozzáadja a gombokat a GameGrid-hez, és eseménykezelőt regisztrál a kattintásokra.



2. DifficultySlider_ValueChanged()
Feladat: Frissíti a nehézségi szint szövegét a csúszka értéke alapján.
```cs
       private void DifficultySlider_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
       {
           // Ellenőrizzük, hogy a DifficultyText létezik-e
           if (DifficultyText != null)
           {
               string[] difficultyLevels = { "Könnyű", "Normál", "Nehéz", "Profi" };
               int level = (int)DifficultySlider.Value;
               DifficultyText.Text = difficultyLevels[level];
           }
       }
```

A csúszka értéke (0-3) alapján kiválasztja a megfelelő szöveget a difficultyLevels tömbből (pl. "Könnyű", "Profi").

Beállítja a DifficultyText szövegét a kiválasztott nehézségi szintre.

3. ShowAnimal(int row, int col)
Feladat: Megjelenít egy állatot egy adott lyukban, majd eltünteti.
```cs
       private void ShowAnimal(int row, int col)
       {
           Button btn = gameButtons[row, col];
           if (btn.Tag.ToString() == "hole")
           {
               double difficulty = DifficultySlider.Value; // 0-3
               double showTime = 2.5 - (difficulty * 0.75); // Könnyű: 2.5s, Profi: 0.25s

               btn.Content = new Image
               {
                   Source = new BitmapImage(new Uri(animalImage)),
                   Width = 90,
                   Height = 90,
                   Stretch = Stretch.Uniform
               };
               btn.Tag = "animal";

               Thread.Sleep(TimeSpan.FromSeconds(showTime));
               System.Diagnostics.Debug.WriteLine($"row: {row}, col: {col}");

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


Ha a gomb Tag-je "hole", lecseréli a képet "állatra" (animalImage).

A nehézségi szint (DifficultySlider) alapján számolja ki az állat megjelenési idejét (nehézség növelése csökkenti az időt).

Probléma: A Thread.Sleep() blokkolja a UI szálat, ami a játék "fagyását" okozza (nem ajánlott WPF-ben).

4. StartGame()
Feladat: Elindítja a játékot.
```cs
       private void StartGame()
       {
           isGameRunning = true;
           score = 0;
           ScoreText.Text = "0";

           while (isGameRunning && score < 10)
           {
               int row = random.Next(0, 3);
               int col = random.Next(0, 3);

               ShowAnimal(row, col);
           }

           isGameRunning = false;
           ResetAnimals();

           if (score >= 10)
           {
               MessageBox.Show("Gratulálok! Nyertél!");
           }
       }
```


Visszaállítja a pontszámot 0-ra és aktiválja a játékot (isGameRunning = true).

Egy while ciklusban véletlenszerűen kiválaszt egy lyukat és meghívja a ShowAnimal() metódust.

Ha a pontszám eléri a 10-et, győzelmi üzenetet jelenít meg.

5. Button_Click()
Feladat: Kezeli a játékos kattintásait.

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

Ha a játék fut (isGameRunning) és a kattintott gomb Tag-je "animal", növeli a pontszámot.

Visszaállítja a lyukat ("hole" kép és Tag).

6. RestartGame()
Feladat: Újraindítja a játékot.

```cs
       private void RestartGame(object sender, RoutedEventArgs e)
       {
           isGameRunning = false;
           StartGame();
       }
```
Leállítja a jelenlegi játékot (isGameRunning = false), majd újraindítja a StartGame()-el.

7. ResetAnimals()
Feladat: Minden lyukat visszaállít alaphelyzetbe.

```cs
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
```

Végigmegy az összes gombon, és ha "animal" a Tag, visszaállítja "hole"-ra.

Fontos Megjegyzések
Szinkron Implementáció: A Thread.Sleep() használata a ShowAnimal() metódusban blokkolja a felhasználói felületet. Ajánlott helyette aszinkron módszerek (pl. async/await + Task.Delay()) vagy időzítők (DispatcherTimer) használata.

Nehézségi Szint: A nehézség növelése gyorsabban tünteti el az állatokat, de a blokkoló Sleep() miatt a játék nehezebbé válhat (a UI nem reagál).

Játéklogika: A while ciklus a StartGame()-ben blokkolja a UI-t, ami nem ideális WPF alkalmazásokban.

Javítási Tipp: Használj DispatcherTimer-t az állatok megjelenítéséhez és eltüntetéséhez, hogy a UI szál ne blokkolódjon!

Kód egyben
```cs
/*
 A szinkron kód használata miatt a fő szál blokkolódik, ami a program lefagyását okozza.
 */

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

namespace Wpf_1_WhacAMole_sync
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
            }
        }

        private void ShowAnimal(int row, int col)
        {
            Button btn = gameButtons[row, col];
            if (btn.Tag.ToString() == "hole")
            {
                double difficulty = DifficultySlider.Value; // 0-3
                double showTime = 2.5 - (difficulty * 0.75); // Könnyű: 2.5s, Profi: 0.25s

                btn.Content = new Image
                {
                    Source = new BitmapImage(new Uri(animalImage)),
                    Width = 90,
                    Height = 90,
                    Stretch = Stretch.Uniform
                };
                btn.Tag = "animal";

                Thread.Sleep(TimeSpan.FromSeconds(showTime));
                System.Diagnostics.Debug.WriteLine($"row: {row}, col: {col}");

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

        private void StartGame()
        {
            isGameRunning = true;
            score = 0;
            ScoreText.Text = "0";

            while (isGameRunning && score < 10)
            {
                int row = random.Next(0, 3);
                int col = random.Next(0, 3);

                ShowAnimal(row, col);
            }

            isGameRunning = false;
            ResetAnimals();

            if (score >= 10)
            {
                MessageBox.Show("Gratulálok! Nyertél!");
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
            isGameRunning = false;
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


