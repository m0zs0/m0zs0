```xaml
<Window x:Class="Wpf_1_coffee_machine.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Wpf_1_coffee_machine"
        mc:Ignorable="d"
         Title="Kávéautomata" Height="600" Width="900">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="3*"/>
            <ColumnDefinition Width="2*"/>
        </Grid.ColumnDefinitions>

        <!-- Bal oldali oszlop - Teljes magasságú kontrollerek -->
        <Grid Grid.Column="0" Margin="10">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="Auto"/>
                <RowDefinition Height="*"/>
            </Grid.RowDefinitions>

            <!-- Érmék és pénzvisszavétel -->
            <GroupBox Grid.Row="0" Header="Elérhető érmék" Margin="0,0,0,10">
                <WrapPanel>
                    <Button Content="50 Ft" Tag="50" Width="80" Margin="5" 
                            PreviewMouseLeftButtonDown="Coin_PreviewMouseLeftButtonDown"/>
                    <Button Content="100 Ft" Tag="100" Width="80" Margin="5" 
                            PreviewMouseLeftButtonDown="Coin_PreviewMouseLeftButtonDown"/>
                    <Button Content="200 Ft" Tag="200" Width="80" Margin="5" 
                            PreviewMouseLeftButtonDown="Coin_PreviewMouseLeftButtonDown"/>
                    <Button x:Name="WithdrawButton" Content="Pénzt kivesz" Width="100" Margin="5"
                            Click="WithdrawButton_Click" Background="#FFD700" IsEnabled="False"/>
                </WrapPanel>
            </GroupBox>

            <!-- Bedobott összeg -->
            <Border Grid.Row="1" BorderBrush="Gray" BorderThickness="1" Background="Transparent" Margin="0,0,0,10" Padding="10" AllowDrop="True" DragEnter="MoneyDropArea_DragEnter" Drop="MoneyDropArea_Drop">
                <StackPanel Orientation="Horizontal" HorizontalAlignment="Center">
                    <TextBlock Text="Bedobott összeg: " FontWeight="Bold"/>
                    <TextBlock x:Name="txtInsertedAmount" Text="0 Ft" Width="80" 
                             TextAlignment="Right" HorizontalAlignment="Right"/>
                </StackPanel>
            </Border>

            <!-- Cukor beállítás -->
            <GroupBox Grid.Row="2" Header="Cukor mennyisége" Margin="0,0,0,10">
                <StackPanel>
                    <Slider x:Name="sldSugar" Minimum="0" Maximum="4" TickPlacement="BottomRight" 
                            TickFrequency="1" IsSnapToTickEnabled="True" Margin="10"
                            ValueChanged="sldSugar_ValueChanged"/>
                    <TextBlock x:Name="txtSugarValue" Text="0" Margin="10" FontSize="16" FontWeight="Bold" VerticalAlignment="Center" HorizontalAlignment="Center"/>
                </StackPanel>
            </GroupBox>

            <!-- Kávék 3x2 rácsban árral -->
            <GroupBox Grid.Row="3" Header="Kávé Választó" Margin="5">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="*"/>
                        <RowDefinition Height="*"/>
                        <RowDefinition Height="*"/>
                    </Grid.RowDefinitions>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="*"/>
                        <ColumnDefinition Width="*"/>
                    </Grid.ColumnDefinitions>

                    <!-- 1. sor -->
                    <StackPanel Grid.Row="0" Grid.Column="0" Orientation="Horizontal" Margin="5">
                        <Button Content="Espresso" Tag="150" Width="100" Height="40" Background="#FFB3E0FF" Click="DrinkButton_Click"/>
                        <TextBlock Text="150 Ft" VerticalAlignment="Center" Margin="10,0"/>
                    </StackPanel>

                    <StackPanel Grid.Row="0" Grid.Column="1" Orientation="Horizontal" Margin="5">
                        <Button Content="Cappuccino" Tag="160" Width="100" Height="40" Background="#FFFFD699" Click="DrinkButton_Click"/>
                        <TextBlock Text="160 Ft" VerticalAlignment="Center" Margin="10,0"/>
                    </StackPanel>

                    <!-- 2. sor -->
                    <StackPanel Grid.Row="1" Grid.Column="0" Orientation="Horizontal" Margin="5">
                        <Button Content="Flat White" Tag="170" Width="100" Height="40" Background="#FFC1E1C1" Click="DrinkButton_Click"/>
                        <TextBlock Text="170 Ft" VerticalAlignment="Center" Margin="10,0"/>
                    </StackPanel>

                    <StackPanel Grid.Row="1" Grid.Column="1" Orientation="Horizontal" Margin="5">
                        <Button Content="Mocha" Tag="180" Width="100" Height="40" Background="#FFFFB3BA" Click="DrinkButton_Click"/>
                        <TextBlock Text="180 Ft" VerticalAlignment="Center" Margin="10,0"/>
                    </StackPanel>

                    <!-- 3. sor -->
                    <StackPanel Grid.Row="2" Grid.Column="0" Orientation="Horizontal" Margin="5">
                        <Button Content="Macchiato" Tag="190" Width="100" Height="40" Background="#FFD4BBDD" Click="DrinkButton_Click"/>
                        <TextBlock Text="190 Ft" VerticalAlignment="Center" Margin="10,0"/>
                    </StackPanel>

                    <StackPanel Grid.Row="2" Grid.Column="1" Orientation="Horizontal" Margin="5">
                        <Button Content="Americano" Tag="200" Width="100" Height="40" Background="#FFFFE699" Click="DrinkButton_Click"/>
                        <TextBlock Text="200 Ft" VerticalAlignment="Center" Margin="10,0"/>
                    </StackPanel>
                </Grid>
            </GroupBox>
        </Grid>

        <!-- Jobb oldali oszlop - Állapotjelző -->
        <Border Grid.Column="1" Margin="10" BorderBrush="Gray" BorderThickness="1">
            <ScrollViewer x:Name="scroller" VerticalScrollBarVisibility="Auto">
                <TextBlock x:Name="txtStatus" Margin="10" TextWrapping="Wrap" 
                         Text="Kérjük, dobjon be pénzt és válasszon italt"/>
            </ScrollViewer>
        </Border>
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

namespace Wpf_1_coffee_machine
{
    public partial class MainWindow : Window
    {
        private int _insertedAmount;
        private const int MaxLines = 50;

        public MainWindow()
        {
            InitializeComponent();
        }

        private void Coin_PreviewMouseLeftButtonDown(object sender, MouseButtonEventArgs e)
        {
            if (sender is Button coin)
            {
                DragDrop.DoDragDrop(coin, coin.Tag.ToString(), DragDropEffects.Copy);
            }
        }

        private void MoneyDropArea_DragEnter(object sender, DragEventArgs e)
        {
            if (!e.Data.GetDataPresent(DataFormats.StringFormat))
                e.Effects = DragDropEffects.None;
        }

        private void MoneyDropArea_Drop(object sender, DragEventArgs e)
        {
            if (int.TryParse(e.Data.GetData(DataFormats.StringFormat).ToString(), out int amount))
            {
                _insertedAmount += amount;
                txtInsertedAmount.Text = $"{_insertedAmount} Ft";
                UpdateStatus($"Bedobott {amount} Ft");
                UpdateWithdrawButtonState();
            }
        }

        private void WithdrawButton_Click(object sender, RoutedEventArgs e)
        {
            if (_insertedAmount > 0)
            {
                UpdateStatus($"Pénzkivétel: {_insertedAmount} Ft");
                _insertedAmount = 0;
                txtInsertedAmount.Text = "0 Ft";
                UpdateWithdrawButtonState();
            }
        }

        private void UpdateWithdrawButtonState()
        {
            WithdrawButton.IsEnabled = _insertedAmount > 0;
        }

        private async void DrinkButton_Click(object sender, RoutedEventArgs e)
        {
            if (sender is Button drinkButton && int.TryParse(drinkButton.Tag.ToString(), out int price))
            {
                if (_insertedAmount < price)
                {
                    UpdateStatus($"Elégtelen összeg! Hiányzik még {price - _insertedAmount} Ft");
                    return;
                }

                _insertedAmount -= price; // Csak levonjuk az árat
                txtInsertedAmount.Text = $"{_insertedAmount} Ft";

                UpdateStatus($"Kiválasztva: {drinkButton.Content} ({price} Ft)");
                UpdateStatus($"Maradék összeg: {_insertedAmount} Ft");
                UpdateWithdrawButtonState();

                await BrewProcessAsync(drinkButton.Content.ToString());

            }
        }

        private void sldSugar_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
        {
            // Cukor érték frissítése a TextBlock-ban
            txtSugarValue.Text = ((int)sldSugar.Value).ToString();

            // Státuszjelző frissítése
            UpdateStatus($"Cukor beállítva: {sldSugar.Value} adag");
        }

        private void UpdateStatus(string message)
        {
            List<string> lines = txtStatus.Text.Split('\n').ToList();
            lines.Add(message);
    
            if(lines.Count > MaxLines)
            {
                lines = lines.Skip(lines.Count - MaxLines).ToList();
            }
    
            txtStatus.Text = string.Join("\n", lines);
            scroller.ScrollToEnd();
        }

        private async Task BrewProcessAsync(string drink)
        {   UpdateStatus($"{drink} elkészítése...");
            await UpdateStatusWithDelay("Darálás...", 2000);
            await UpdateStatusWithDelay("Vízmelegítés...", 3000);
            await UpdateStatusWithDelay("Főzés...", 4000);
            if (int.Parse(txtSugarValue.Text) > 0) await UpdateStatusWithDelay($"Cukor adagolása ({txtSugarValue.Text} adag)...", 1000);
            await UpdateStatusWithDelay("Kiadás...", 2000);
            txtStatus.Text += $"\n{drink} kész! Egészségedre!";
        }

        private async Task UpdateStatusWithDelay(string message, int delay)
        {
            await Task.Delay(delay);
            UpdateStatus(message);
        }
    }
    
}
```
