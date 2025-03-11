```xaml
<Grid>
    <StackPanel>
        <Button x:Name="StartButton" Content="Start" Click="StartButton_Click" Width="100" Height="30" Margin="10"/>
        <TextBlock x:Name="ResultText" Text="Nyomd meg a gombot!" FontSize="16" VerticalAlignment="Center" HorizontalAlignment="Center"/>
    </StackPanel>
</Grid>
```


# 1. Szinkron verzió (Thread.Sleep)

```cs
private void StartButton_Click(object sender, RoutedEventArgs e)
{
    StartButton.IsEnabled = false;
    ResultText.Text = "Feldolgozás...";

    // Szinkron módon várakozunk 3 másodpercet
    Thread.Sleep(3000);

    ResultText.Text = "Kész! A művelet befejeződött.";
    StartButton.IsEnabled = true;
}
```

Mi történik?

Amikor megnyomjuk a gombot, a StartButton_Click metódus végrehajtódik.

A Thread.Sleep(3000) blokkolja az aktuális szálat – ebben az esetben az UI szálat –, amely a WPF alkalmazásban minden grafikus elem (pl. gombok, ablak mozgatása) kezeléséért felel.

3 másodpercig az alkalmazás "lefagy": nem lehet mozgatni az ablakot, nem reagál semmire, mert az UI szál várakozik.

Hogyan működik?

A Thread.Sleep egy szinkron függvény, amely az aktuális szál végrehajtását felfüggeszti a megadott időre.

Mivel az UI szál blokkolva van, az alkalmazás nem tudja frissíteni a képernyőt, és nem reagál a felhasználói interakciókra.

Tesztelés:

Nyomd meg a gombot, és próbáld meg mozgatni az ablakot. Nem fog sikerülni, amíg a 3 másodperc le nem telik.



# 2. Aszinkron verzió (async/await)

```cs
private async void StartButton_Click(object sender, RoutedEventArgs e)
{
    StartButton.IsEnabled = false;
    ResultText.Text = "Feldolgozás...";

    // Aszinkron módon várakozunk 3 másodpercet
    await Task.Delay(3000);

    ResultText.Text = "Kész! A művelet befejeződött.";
    StartButton.IsEnabled = true;
}
```

Mi történik?

A gomb megnyomásakor a StartButton_Click metódus elindul, és az await Task.Delay(3000)-nél a vezérlés visszaadódik az UI szálnak.

A Task.Delay egy aszinkron művelet, amely nem blokkolja az UI szálat, hanem egy háttérfeladatként fut.

Amíg a 3 másodperc várakozás tart, az ablak mozgatható marad, és az alkalmazás reszponzív.

A várakozás alatt a kód folytatódik, és frissíti a szöveget.

Hogyan működik?

Az async kulcsszó jelzi, hogy a metódus aszinkron módon fut.

Az await kulcsszó megvárja a Task.Delay befejezését, de közben nem blokkolja az UI szálat. A vezérlés visszaadódik az úgynevezett "hívó kontextushoz" (itt az UI szálhoz), és a folytatás automatikusan az UI szálon történik a Task befejezése után.

Ez a mechanizmus a .NET Task Parallel Library (TPL)-jére épül.

Tesztelés

Nyomd meg a gombot, és próbáld meg mozgatni az ablakot. Sikerülni fog.
