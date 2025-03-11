# I. Drag & Drop programozása:
[1] Indítás: A PreviewMouseLeftButtonDown elindítja a drag műveletet az érme értékével.
[2] Ellenőrzés: A DragEnter biztosítja, hogy csak megfelelő adat dobható a területre.
[3] Lezárás: A Drop megkapja az adatot, és feldolgozza (pl. hozzáadja az összeghez).



XAML:

-Három gomb: "10", "20", "50" Content-tel.

```xaml
<Button Content="10" Width="50" Margin="5" PreviewMouseLeftButtonDown="Button_PreviewMouseLeftButtonDown"/>
```

-Egy Border, benne egy TextBlock, ami kezdetben "összeg = 0"-t mutat.

```xaml
<Border Grid.Row="1" Margin="10" BorderBrush="Black" BorderThickness="1" Background="LightGray"
                AllowDrop="True" DragEnter="DropArea_DragEnter" Drop="DropArea_Drop">
  <TextBlock x:Name="infoTextBlock" Text="sum = 0" FontSize="20" HorizontalAlignment="Center" VerticalAlignment="Center"/>
</Border>
```



XAML.CS:

[1] Egérgomb lenyomása Gomb-on (PreviewMouseLeftButtonDown): 
- Ha rákattintasz egy gombra (pl. "20"), elindul a drag művelet a gomb Content-jével (itt "20").
- A PreviewMouseLeftButtonDown a normál MouseLeftButtonDown előtt fut le, így még azelőtt elkapod az eseményt, hogy a gomb alapértelmezett viselkedése (pl. kattintás) érvényesülne.

```cs
private void Button_PreviewMouseLeftButtonDown(object sender, MouseButtonEventArgs e)
{
    if (sender is Button button)
    {
        // A gomb Content-jét visszük át
        DragDrop.DoDragDrop(button, button.Content.ToString(), DragDropEffects.Copy);
    }
}
```

Ez az esemény akkor fut le, amikor a bal egérgombot lenyomod egy gombon (pl. "50").

A sender a gomb, amit megnyomtak, a Content-ben tároljuk az értékét.

A DragDrop.DoDragDrop metódus elindítja a drag műveletet:

- Első paraméter (button): Az az objektum, amit "megragadsz" (a gomb).

- Második paraméter (button.Content.ToString()): Az az adat, amit átviszel (itt a Content értéke, pl. "50").

- Harmadik paraméter (DragDropEffects.Copy): Azt jelzi, hogy az adat másolódik, nem mozdul el az eredeti helyéről.



[2] Célterület fölé lépés (DragEnter): Ellenőrzi, hogy az átvitt adat string-e. Ha nem, tiltja a droppolást.

```cs
private void DropArea_DragEnter(object sender, DragEventArgs e)
{
    // Ellenőrizzük, hogy stringet kaptunk-e
    if (!e.Data.GetDataPresent(DataFormats.StringFormat))
    {
        e.Effects = DragDropEffects.None;
    }
}
```

Ez az esemény akkor fut le, amikor egy "draggelt" objektum (most a Content értéke) belép a drop célterület (a Border) fölé.

Ellenőrzi, hogy az átvitt adat string formátumú-e (DataFormats.StringFormat). Ha nem, akkor a DragDropEffects.None-nal jelzi, hogy ide nem lehet dobni.

Ez egyfajta szűrő: csak akkor engedi a droppolást, ha az adat megfelelő formátumú.


[3] Eldobás (Drop): Ha a Border-re dobod a gombot, az átvitt érték (pl. "20") hozzáadódik a sum-hoz, és a TextBlock frissül (pl. "összeg = 20").

```cs
private void DropArea_Drop(object sender, DragEventArgs e)
{
    // Lekérjük az átvitt adatot és hozzáadjuk a sum-hoz
    if (int.TryParse(e.Data.GetData(DataFormats.StringFormat).ToString(), out int value))
    {
        sum += value;
        infoTextBlock.Text = $"összeg = {sum}";
    }
}
```

Ez akkor fut le, amikor az egérgombot felengeded a célterület (a Border) felett.

Az e.Data.GetData lekéri az átvitt adatot (pl. "50"), amit a DragDrop.DoDragDrop korábban átadott.

Az int.TryParse átalakítja ezt az értéket számmá (value), és ha sikerül, hozzáadja a sum-hoz, és frissíti a szöveget.

```XAML
<Window x:Class="DragDropExample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Drag & Drop Példa" Height="300" Width="400">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Gombok -->
        <StackPanel Orientation="Horizontal" Margin="10">
            <Button Content="10" Width="50" Margin="5" PreviewMouseLeftButtonDown="Button_PreviewMouseLeftButtonDown"/>
            <Button Content="20" Width="50" Margin="5" PreviewMouseLeftButtonDown="Button_PreviewMouseLeftButtonDown"/>
            <Button Content="50" Width="50" Margin="5" PreviewMouseLeftButtonDown="Button_PreviewMouseLeftButtonDown"/>
        </StackPanel>

        <!-- Drop terület -->
        <Border Grid.Row="1" Margin="10" BorderBrush="Black" BorderThickness="1" Background="LightGray"
                AllowDrop="True" DragEnter="DropArea_DragEnter" Drop="DropArea_Drop" DragOver="DropArea_DragOver">
            <TextBlock x:Name="txtSum" Text="sum = 0" FontSize="20" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </Border>
    </Grid>
</Window>

```

Code behind
```cs
namespace DragDropExample
{
    public partial class MainWindow : Window
    {
        private int sum = 0;

        public MainWindow()
        {
            InitializeComponent();
        }

        private void Button_PreviewMouseLeftButtonDown(object sender, MouseButtonEventArgs e)
        {
            if (sender is Button button)
            {
                // A gomb Content-jét átvisszük
                DragDrop.DoDragDrop(button, button.Content.ToString(), DragDropEffects.Copy);
            }
        }

        private void DropArea_DragEnter(object sender, DragEventArgs e)
        {
            // Ellenőrizzük, hogy stringet kaptunk-e
            if (!e.Data.GetDataPresent(DataFormats.StringFormat))
            {
                e.Effects = DragDropEffects.None;
            }
        }

        private void DropArea_Drop(object sender, DragEventArgs e)
        {
            // Lekérjük az átvitt adatot és hozzáadjuk a sum-hoz
            if (int.TryParse(e.Data.GetData(DataFormats.StringFormat).ToString(), out int value))
            {
                sum += value;
                txtSum.Text = $"sum = {sum}";
            }
        }
    }
```



II. Fejlesszük tovább a drag&drop élményt, és adjunk vizuális visszajelzést, ami az egérrel mozog vonszolás közben (Content).

A WPF-ben a drag&drop események közül a következők a leggyakoribbak:
- PreviewMouseLeftButtonDown: Már használjuk a drag indítására.
- DragEnter: Már használjuk a drop terület ellenőrzésére.
- Drop: Már használjuk az adat fogadására.
- DragOver: Ezt most hozzáadjuk! Akkor fut le, amikor a draggelt elem a célterület felett mozog. Használhatjuk a vizuális visszajelzés frissítésére (pl. az egér pozíciójának követésére).
- DragLeave: Akkor fut le, amikor a draggelt elem elhagyja a célterületet. Ezt is bemutathatjuk, ha pl. a vizuális visszajelzést el akarjuk tüntetni.

```xaml
<Window x:Class="Wpf_1_draganddrop.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Wpf_1_draganddrop"
        mc:Ignorable="d"
        Title="Drag and Drop" Height="450" Width="800"
        AllowDrop="True" DragOver="Window_DragOver">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <!-- Gombok -->
        <StackPanel Orientation="Horizontal" Margin="10">
            <Button Content="10" Width="50" Margin="5" PreviewMouseLeftButtonDown="Button_PreviewMouseLeftButtonDown"/>
            <Button Content="20" Width="50" Margin="5" PreviewMouseLeftButtonDown="Button_PreviewMouseLeftButtonDown"/>
            <Button Content="50" Width="50" Margin="5" PreviewMouseLeftButtonDown="Button_PreviewMouseLeftButtonDown"/>
        </StackPanel>

        <!-- Drop terület -->
        <Border Grid.Row="1" Margin="10" BorderBrush="Black" BorderThickness="1" Background="LightGray"
                AllowDrop="True" DragEnter="DropArea_DragEnter" Drop="DropArea_Drop" DragLeave="DropArea_DragLeave"/>

        <!-- TextBlock a drop területen belül -->
        <TextBlock Grid.Row="1" x:Name="txtSum" Text="sum = 0" FontSize="20" HorizontalAlignment="Center" VerticalAlignment="Center" IsHitTestVisible="False"/>

        <!-- Canvas az egész ablakon -->
        <Canvas x:Name="dragCanvas" Grid.Row="0" Grid.RowSpan="2"/>
    </Grid>
</Window>
```

**Ablak definíciója:**
Az AllowDrop="True" és a DragOver="Window_DragOver" azt jelzi, hogy az egész ablak fogadhat drag&drop eseményeket, és a Window_DragOver metódus kezeli az egér mozgását a vonszolás alatt. Ez lehetővé teszi, hogy a dragVisual az ablak teljes területén simán mozogjon.
**Grid elrendezés:**
Két sor van definiálva: az első (Height="Auto") a gombok számára, a második (Height="*") a drop terület és a TextBlock számára, így az alsó sor kitölti a maradék helyet.
**Gombok (StackPanel):**
Három gomb (10, 20, 50) vízszintesen rendezve, mindegyikhez kötve a PreviewMouseLeftButtonDown esemény, amely elindítja a drag műveletet.
**Drop terület (Border):**
A Grid.Row="1"-ben helyezkedik el, AllowDrop="True"-val, és három eseménykezelője van: DragEnter (belépéskor), Drop (eldobáskor), DragLeave (kilépéskor). Ez a terület felelős a számok fogadásáért és a vizuális visszajelzésért (zöld háttér).
**TextBlock (txtSum):**
A sum értékét jeleníti meg, a Grid.Row="1"-ben középre igazítva. Az IsHitTestVisible="False" biztosítja, hogy ne zavarja a Border drag&drop eseményeit, így a drop a TextBlock felett is működik.
**Canvas (dragCanvas):**
Az egész ablakot lefedi (Grid.Row="0" Grid.RowSpan="2"), és a legutolsó elemként van definiálva a Grid-ben, ezért a legmagasabb Z-indexszel rendelkezik. Ez azt jelenti, hogy a dragVisual (a vonszolt szám) minden más elem fölött jelenik meg. Miért van szükség a Canvas-ra? Mert szabadon pozícionálhatjuk a dragVisual-t az egérhez igazítva, amit más elrendezési panelek (pl. Grid, StackPanel) nem tesznek lehetővé ilyen egyszerűen. Miért van felül? Mert a Grid-ben az utolsó elemként definiált, így a Z-rendben a legfelső réteg, garantálva, hogy a dragVisual látható maradjon a Border és a TextBlock felett.


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

namespace Wpf_1_draganddrop
{
    public partial class MainWindow : Window
    {
        private int sum = 0;
        private TextBlock dragVisual;
        private readonly SolidColorBrush defaultBrush = Brushes.OrangeRed; // Alapértelmezett háttér
        private readonly SolidColorBrush overBrush = Brushes.LightGreen;  // Border felett zöld

        public MainWindow()
        {
            InitializeComponent();
        }

        private void Button_PreviewMouseLeftButtonDown(object sender, MouseButtonEventArgs e)
        {
            if (sender is Button button)
            {
                dragVisual = new TextBlock
                {
                    Text = button.Content.ToString(),
                    FontSize = 16,
                    Foreground = Brushes.Black,
                    Background = defaultBrush,
                    Padding = new Thickness(5),
                    Opacity = 0.8
                };

                dragCanvas.Children.Add(dragVisual);

                // Kezdeti pozíció beállítása
                Point initialPosition = e.GetPosition(dragCanvas);
                Canvas.SetLeft(dragVisual, initialPosition.X + 5);
                Canvas.SetTop(dragVisual, initialPosition.Y + 5);

                // Drag indítása az ablakon keresztül
                DragDrop.DoDragDrop(button, button.Content.ToString(), DragDropEffects.Copy);

                // Drag végeztével cleanup
                dragCanvas.Children.Remove(dragVisual);
                dragVisual = null;
            }
        }

        private void Window_DragOver(object sender, DragEventArgs e)
        {
            if (dragVisual != null)
            {
                // Az egér pozícióját az ablakhoz viszonyítva követjük
                Point mousePosition = e.GetPosition(dragCanvas);
                Canvas.SetLeft(dragVisual, mousePosition.X + 5);
                Canvas.SetTop(dragVisual, mousePosition.Y - 25);
                System.Diagnostics.Debug.WriteLine($"Mouse: {mousePosition.X}, {mousePosition.Y}");
            }
        }

        private void DropArea_DragEnter(object sender, DragEventArgs e)
        {
            if (!e.Data.GetDataPresent(DataFormats.StringFormat))
            {
                e.Effects = DragDropEffects.None;
            }
            else if (dragVisual != null)
            {
                // Zöldre váltás, amikor belép a Border fölé
                dragVisual.Background = overBrush;
                e.Effects = DragDropEffects.Copy;  // Egyértelműen jelezzük, hogy droppable
            }
        }

        private void DropArea_DragLeave(object sender, DragEventArgs e)
        {
            if (dragVisual != null)
            {
                // Visszaáll pirosra, amikor elhagyja a Border-t
                dragVisual.Background = defaultBrush;
            }
        }

        private void DropArea_Drop(object sender, DragEventArgs e)
        {
            if (int.TryParse(e.Data.GetData(DataFormats.StringFormat).ToString(), out int value))
            {
                sum += value;
                txtSum.Text = $"sum = {sum}";
            }

            if (dragVisual != null)
            {
                dragCanvas.Children.Remove(dragVisual);
                dragVisual = null;
            }
        }
    }
}
```

**Változók:**
sum: A droppolt számok összege, amit a txtSum TextBlock mutat.
dragVisual: A vonszolt számot megjelenítő TextBlock, amit dinamikusan hozunk létre és távolítunk el.
defaultBrush és overBrush: Színváltozók az alapértelmezett (narancsvörös) és a Border feletti (világoszöld) háttérhez.
**Button_PreviewMouseLeftButtonDown:**
Amikor egy gombra kattintasz, létrehozza a dragVisual-t a gomb tartalmával (pl. "10"), narancsvörös háttérrel, és az egér kezdeti pozíciójához igazítja (initialPosition.X + 5, Y + 5). A DragDrop.DoDragDrop elindítja a drag műveletet, az adat pedig a gomb szövege. A drag végeztével a dragVisual-t eltávolítjuk a Canvas-ról. Miért kell a Canvas? Mert itt adjuk hozzá a dragVisual-t, és a Canvas abszolút pozícionálási képessége lehetővé teszi, hogy az egérhez igazítsuk.
**Window_DragOver:**
Az ablakszintű DragOver esemény folyamatosan fut, amíg a drag művelet tart, és frissíti a dragVisual pozícióját az egérhez képest (X + 5, Y - 25). A Y - 25 eltolás valószínűleg azt szolgálja, hogy a szám az egérkurzor felett jelenjen meg, ne alatta. A Debug.WriteLine segít ellenőrizni az egér pozícióját. Miért kell a Canvas? Mert az e.GetPosition(dragCanvas) a Canvas koordinátarendszerét használja, ami az egész ablakot lefedi, így az egérkövetés az ablak bármely pontján működik.
**DropArea_DragEnter:**
Amikor az egér a Border fölé lép, ellenőrzi, hogy az átvitt adat string-e. Ha igen, a dragVisual háttere zöldre (overBrush) vált, és az e.Effects = DragDropEffects.Copy jelzi, hogy a drop engedélyezett.
DropArea_DragLeave:
Amikor az egér elhagyja a Border-t, a dragVisual háttere visszaáll narancsvörösre (defaultBrush).
**DropArea_Drop:**
Ha a Border-re dobod a számot, az átvitt stringet számmá alakítja (int.TryParse), hozzáadja a sum-hoz, frissíti a txtSum szövegét, majd eltávolítja a dragVisual-t. A TextBlock IsHitTestVisible="False" tulajdonsága miatt ez a Border bármely pontján működik.
**Miért van a Canvas felül?**
A dragCanvas a Grid utolsó elemeként van definiálva, így a legmagasabb Z-indexszel rendelkezik. Ez biztosítja, hogy a dragVisual a Border, a TextBlock és a gombok felett maradjon, különben a vonszolt szám eltakaródna, amikor ezek fölött mozog.
