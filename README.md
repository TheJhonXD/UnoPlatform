# UnoPlatform

Formulario para agregar un nuevo producto
``` xml
<Page
    x:Class="MyApp.AddProductPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center" Width="300">

            <TextBlock Text="Agregar Producto" FontSize="24" Margin="0,0,0,20" HorizontalAlignment="Center"/>

            <!-- Input para el nombre del producto -->
            <TextBox x:Name="ProductNameTextBox" PlaceholderText="Nombre del producto" Margin="0,0,0,10"/>

            <!-- Input para el precio del producto -->
            <TextBox x:Name="ProductPriceTextBox" PlaceholderText="Precio del producto" Margin="0,0,0,10" InputScope="Number"/>

            <!-- Select para la categoría del producto -->
            <ComboBox x:Name="CategoryComboBox" PlaceholderText="Seleccionar categoría" Margin="0,0,0,10">
                <ComboBoxItem Content="Electrónica"/>
                <ComboBoxItem Content="Ropa"/>
                <ComboBoxItem Content="Hogar"/>
                <ComboBoxItem Content="Libros"/>
            </ComboBox>

            <!-- Select para el proveedor del producto -->
            <ComboBox x:Name="SupplierComboBox" PlaceholderText="Seleccionar proveedor" Margin="0,0,0,10">
                <ComboBoxItem Content="Proveedor 1"/>
                <ComboBoxItem Content="Proveedor 2"/>
                <ComboBoxItem Content="Proveedor 3"/>
            </ComboBox>

            <!-- Input para la URL del producto -->
            <TextBox x:Name="ProductUrlTextBox" PlaceholderText="URL del producto" Margin="0,0,0,10"/>

            <!-- TextArea para la descripción del producto -->
            <TextBox x:Name="ProductDescriptionTextBox" PlaceholderText="Descripción del producto" Margin="0,0,0,20" AcceptsReturn="True" TextWrapping="Wrap" Height="100"/>

            <!-- Botón para agregar el producto -->
            <Button Content="Agregar Producto" Click="OnAddProductButtonClick" HorizontalAlignment="Center"/>
        </StackPanel>
    </Grid>
</Page>

```

``` c#
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;

namespace MyApp
{
    public sealed partial class AddProductPage : Page
    {
        public AddProductPage()
        {
            this.InitializeComponent();
        }

        private void OnAddProductButtonClick(object sender, RoutedEventArgs e)
        {
            // Capturar los datos ingresados en los inputs
            string productName = ProductNameTextBox.Text;
            string productPriceText = ProductPriceTextBox.Text;
            string category = (CategoryComboBox.SelectedItem as ComboBoxItem)?.Content.ToString();
            string supplier = (SupplierComboBox.SelectedItem as ComboBoxItem)?.Content.ToString();
            string productUrl = ProductUrlTextBox.Text;
            string productDescription = ProductDescriptionTextBox.Text;

            // Validar y convertir el precio
            if (double.TryParse(productPriceText, out double productPrice))
            {
                // Lógica para agregar el producto (por ejemplo, guardarlo en una base de datos)
                // Aquí podrías llamar a un servicio o agregar el producto a una lista.

                // Mostrar mensaje de éxito
                ContentDialog successDialog = new ContentDialog
                {
                    Title = "Producto Agregado",
                    Content = "El producto ha sido agregado exitosamente.",
                    CloseButtonText = "Ok"
                };

                _ = successDialog.ShowAsync();
            }
            else
            {
                // Mostrar mensaje de error si el precio no es válido
                ContentDialog errorDialog = new ContentDialog
                {
                    Title = "Error",
                    Content = "Por favor, ingrese un precio válido.",
                    CloseButtonText = "Ok"
                };

                _ = errorDialog.ShowAsync();
            }
        }
    }
}

```

## Con API
``` c#
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json;

namespace MyApp.Services
{
    public class ApiService
    {
        private readonly HttpClient _httpClient;

        public ApiService()
        {
            _httpClient = new HttpClient();
        }

        public async Task<List<string>> GetCategoriesAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/categorias");
            var categories = JsonConvert.DeserializeObject<List<string>>(response);
            return categories;
        }

        public async Task<List<string>> GetSuppliersAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/proveedores");
            var suppliers = JsonConvert.DeserializeObject<List<string>>(response);
            return suppliers;
        }
    }
}

```
``` c#
using MyApp.Services;
using System.Collections.Generic;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;

namespace MyApp
{
    public sealed partial class AddProductPage : Page
    {
        private readonly ApiService _apiService;

        public AddProductPage()
        {
            this.InitializeComponent();
            _apiService = new ApiService();
        }

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);

            // Obtener y poblar categorías
            List<string> categories = await _apiService.GetCategoriesAsync();
            foreach (var category in categories)
            {
                CategoryComboBox.Items.Add(new ComboBoxItem { Content = category });
            }

            // Obtener y poblar proveedores
            List<string> suppliers = await _apiService.GetSuppliersAsync();
            foreach (var supplier in suppliers)
            {
                SupplierComboBox.Items.Add(new ComboBoxItem { Content = supplier });
            }
        }

        private void OnAddProductButtonClick(object sender, Windows.UI.Xaml.RoutedEventArgs e)
        {
            string productName = ProductNameTextBox.Text;
            string productPriceText = ProductPriceTextBox.Text;
            string category = (CategoryComboBox.SelectedItem as ComboBoxItem)?.Content.ToString();
            string supplier = (SupplierComboBox.SelectedItem as ComboBoxItem)?.Content.ToString();
            string productUrl = ProductUrlTextBox.Text;
            string productDescription = ProductDescriptionTextBox.Text;

            if (double.TryParse(productPriceText, out double productPrice))
            {
                // Lógica para agregar el producto
                ContentDialog successDialog = new ContentDialog
                {
                    Title = "Producto Agregado",
                    Content = "El producto ha sido agregado exitosamente.",
                    CloseButtonText = "Ok"
                };
                _ = successDialog.ShowAsync();
            }
            else
            {
                ContentDialog errorDialog = new ContentDialog
                {
                    Title = "Error",
                    Content = "Por favor, ingrese un precio válido.",
                    CloseButtonText = "Ok"
                };
                _ = errorDialog.ShowAsync();
            }
        }
    }
}

```
## Agregar Post
``` c#
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;

namespace MyApp.Services
{
    public class ApiService
    {
        private readonly HttpClient _httpClient;

        public ApiService()
        {
            _httpClient = new HttpClient();
        }

        public async Task<List<string>> GetCategoriesAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/categorias");
            var categories = JsonConvert.DeserializeObject<List<string>>(response);
            return categories;
        }

        public async Task<List<string>> GetSuppliersAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/proveedores");
            var suppliers = JsonConvert.DeserializeObject<List<string>>(response);
            return suppliers;
        }

        public async Task<bool> AddProductAsync(Product product)
        {
            var json = JsonConvert.SerializeObject(product);
            var content = new StringContent(json, Encoding.UTF8, "application/json");

            var response = await _httpClient.PostAsync("https://api.ejemplo.com/productos", content);

            return response.IsSuccessStatusCode;
        }
    }

    public class Product
    {
        public string Name { get; set; }
        public double Price { get; set; }
        public string Category { get; set; }
        public string Supplier { get; set; }
        public string Url { get; set; }
        public string Description { get; set; }
    }
}

```

``` c#
using MyApp.Services;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;

namespace MyApp
{
    public sealed partial class AddProductPage : Page
    {
        private readonly ApiService _apiService;

        public AddProductPage()
        {
            this.InitializeComponent();
            _apiService = new ApiService();
        }

        protected override async void OnNavigatedTo(Windows.UI.Xaml.Navigation.NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);

            // Obtener y poblar categorías
            List<string> categories = await _apiService.GetCategoriesAsync();
            foreach (var category in categories)
            {
                CategoryComboBox.Items.Add(new ComboBoxItem { Content = category });
            }

            // Obtener y poblar proveedores
            List<string> suppliers = await _apiService.GetSuppliersAsync();
            foreach (var supplier in suppliers)
            {
                SupplierComboBox.Items.Add(new ComboBoxItem { Content = supplier });
            }
        }

        private async void OnAddProductButtonClick(object sender, RoutedEventArgs e)
        {
            // Capturar los datos del formulario
            string productName = ProductNameTextBox.Text;
            string productPriceText = ProductPriceTextBox.Text;
            string category = (CategoryComboBox.SelectedItem as ComboBoxItem)?.Content.ToString();
            string supplier = (SupplierComboBox.SelectedItem as ComboBoxItem)?.Content.ToString();
            string productUrl = ProductUrlTextBox.Text;
            string productDescription = ProductDescriptionTextBox.Text;

            // Validar el precio
            if (double.TryParse(productPriceText, out double productPrice))
            {
                // Crear el objeto producto
                var product = new Product
                {
                    Name = productName,
                    Price = productPrice,
                    Category = category,
                    Supplier = supplier,
                    Url = productUrl,
                    Description = productDescription
                };

                // Enviar el producto a la API
                bool success = await _apiService.AddProductAsync(product);

                if (success)
                {
                    // Mostrar mensaje de éxito
                    ContentDialog successDialog = new ContentDialog
                    {
                        Title = "Producto Agregado",
                        Content = "El producto ha sido agregado exitosamente.",
                        CloseButtonText = "Ok"
                    };
                    _ = successDialog.ShowAsync();
                }
                else
                {
                    // Mostrar mensaje de error
                    ContentDialog errorDialog = new ContentDialog
                    {
                        Title = "Error",
                        Content = "No se pudo agregar el producto. Inténtalo de nuevo.",
                        CloseButtonText = "Ok"
                    };
                    _ = errorDialog.ShowAsync();
                }
            }
            else
            {
                // Mostrar mensaje de error si el precio no es válido
                ContentDialog errorDialog = new ContentDialog
                {
                    Title = "Error",
                    Content = "Por favor, ingrese un precio válido.",
                    CloseButtonText = "Ok"
                };
                _ = errorDialog.ShowAsync();
            }
        }
    }
}

```

## Traer una lista de productos
``` c#
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json;

namespace MyApp.Services
{
    public class ApiService
    {
        private readonly HttpClient _httpClient;

        public ApiService()
        {
            _httpClient = new HttpClient();
        }

        public async Task<List<Product>> GetProductsAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/productos");
            var products = JsonConvert.DeserializeObject<List<Product>>(response);
            return products;
        }

        public async Task<List<string>> GetCategoriesAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/categorias");
            var categories = JsonConvert.DeserializeObject<List<string>>(response);
            return categories;
        }

        public async Task<List<string>> GetSuppliersAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/proveedores");
            var suppliers = JsonConvert.DeserializeObject<List<string>>(response);
            return suppliers;
        }

        public async Task<bool> AddProductAsync(Product product)
        {
            var json = JsonConvert.SerializeObject(product);
            var content = new StringContent(json, Encoding.UTF8, "application/json");

            var response = await _httpClient.PostAsync("https://api.ejemplo.com/productos", content);

            return response.IsSuccessStatusCode;
        }
    }

    public class Product
    {
        public string Name { get; set; }
        public double Price { get; set; }
        public string Category { get; set; }
        public string Supplier { get; set; }
        public string Url { get; set; }
        public string Description { get; set; }
    }
}

```

``` xaml
<Page
    x:Class="MyApp.ProductListPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <StackPanel>
            <TextBlock Text="Lista de Productos" FontSize="24" Margin="20,0,0,20" HorizontalAlignment="Center"/>

            <!-- ListView para mostrar la lista de productos -->
            <ListView x:Name="ProductsListView">
                <ListView.ItemTemplate>
                    <DataTemplate x:DataType="local:Product">
                        <StackPanel Orientation="Vertical" Padding="10">
                            <TextBlock Text="{x:Bind Name}" FontSize="18"/>
                            <TextBlock Text="{x:Bind Category}" FontSize="14" Foreground="Gray"/>
                            <TextBlock Text="{x:Bind Price, StringFormat='{}{0:C}'}" FontSize="16"/>
                            <TextBlock Text="{x:Bind Supplier}" FontSize="14"/>
                            <TextBlock Text="{x:Bind Url}" FontSize="14" Foreground="Blue"/>
                            <TextBlock Text="{x:Bind Description}" FontSize="14" TextWrapping="WrapWholeWords"/>
                        </StackPanel>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </StackPanel>
    </Grid>
</Page>

```

``` c#
using MyApp.Services;
using System.Collections.Generic;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;

namespace MyApp
{
    public sealed partial class ProductListPage : Page
    {
        private readonly ApiService _apiService;

        public ProductListPage()
        {
            this.InitializeComponent();
            _apiService = new ApiService();
        }

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);

            // Obtener la lista de productos desde la API
            List<Product> products = await _apiService.GetProductsAsync();

            // Asignar la lista de productos al ListView
            ProductsListView.ItemsSource = products;
        }
    }
}

```
## Datos de combobox desde API

``` c#
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json;

namespace MyApp.Services
{
    public class ApiService
    {
        private readonly HttpClient _httpClient;

        public ApiService()
        {
            _httpClient = new HttpClient();
        }

        public async Task<List<string>> GetCategoriesAsync()
        {
            var response = await _httpClient.GetStringAsync("https://api.ejemplo.com/categorias");
            var categories = JsonConvert.DeserializeObject<List<string>>(response);
            return categories;
        }
    }
}

```


``` xaml
<Page
    x:Class="MyApp.AddProductPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <StackPanel>
            <TextBlock Text="Seleccione una Categoría" FontSize="18" Margin="20"/>
            
            <!-- ComboBox para mostrar categorías -->
            <ComboBox x:Name="CategoryComboBox" Header="Categoría" PlaceholderText="Seleccione una categoría"/>
        </StackPanel>
    </Grid>
</Page>

```

### File.xaml.cs

``` c#
using MyApp.Services;
using System.Collections.Generic;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;

namespace MyApp
{
    public sealed partial class AddProductPage : Page
    {
        private readonly ApiService _apiService;

        public AddProductPage()
        {
            this.InitializeComponent();
            _apiService = new ApiService();
        }

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);

            // Obtener las categorías desde la API
            List<string> categories = await _apiService.GetCategoriesAsync();

            // Agregar cada categoría al ComboBox
            foreach (var category in categories)
            {
                CategoryComboBox.Items.Add(new ComboBoxItem { Content = category });
            }
        }
    }
}

```

## Menu de navegacion

``` xaml
<Page
    x:Class="MyApp.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <!-- Menú de navegación -->
        <Grid HorizontalAlignment="Stretch" VerticalAlignment="Top" Background="LightGray" Height="60">
            <!-- Definir las columnas para el layout -->
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/> <!-- Columna para el logo/nombre -->
                <ColumnDefinition Width="*"/>    <!-- Columna para espaciar el contenido -->
                <ColumnDefinition Width="Auto"/> <!-- Columna para las opciones de navegación -->
            </Grid.ColumnDefinitions>

            <!-- Logo o Nombre -->
            <StackPanel Grid.Column="0" Orientation="Horizontal" VerticalAlignment="Center" Margin="10">
                <Image Source="ms-appx:///Assets/Logo.png" Height="40" Width="40" Margin="0,0,10,0"/>
                <TextBlock Text="MyApp" FontSize="24" VerticalAlignment="Center"/>
            </StackPanel>

            <!-- Opciones de navegación alineadas a la derecha -->
            <StackPanel Grid.Column="2" Orientation="Horizontal" VerticalAlignment="Center" HorizontalAlignment="Right" Margin="10">
                <Button Content="Inicio" Style="{StaticResource NavigationButtonStyle}" Margin="10,0"/>
                <Button Content="Administrar" Style="{StaticResource NavigationButtonStyle}" Margin="10,0"/>
                <Button Content="Buscar" Style="{StaticResource NavigationButtonStyle}" Margin="10,0"/>
            </StackPanel>
        </Grid>

        <!-- Contenido principal -->
        <Grid>
            <!-- Aquí irá el contenido de la página -->
        </Grid>
    </Grid>
</Page>

```


``` xaml
<Application.Resources>
    <Style x:Key="NavigationButtonStyle" TargetType="Button">
        <Setter Property="Background" Value="Transparent"/>
        <Setter Property="Foreground" Value="Black"/>
        <Setter Property="BorderBrush" Value="Transparent"/>
        <Setter Property="FontSize" Value="16"/>
        <Setter Property="Padding" Value="10,5"/>
        <Setter Property="HorizontalAlignment" Value="Center"/>
        <Setter Property="VerticalAlignment" Value="Center"/>
        <Setter Property="Cursor" Value="Hand"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <ContentPresenter HorizontalAlignment="Center" VerticalAlignment="Center"/>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</Application.Resources>

```

## NavigationView

``` xaml
<Page
    x:Class="MyApp.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <!-- NavigationView para el menú de navegación -->
        <NavigationView
            x:Name="NavView"
            PaneDisplayMode="Top"
            IsBackButtonVisible="Collapsed">

            <!-- Cabecera: Logo y Nombre de la aplicación -->
            <NavigationView.Header>
                <StackPanel Orientation="Horizontal" VerticalAlignment="Center">
                    <Image Source="ms-appx:///Assets/Logo.png" Height="40" Width="40" Margin="0,0,10,0"/>
                    <TextBlock Text="MyApp" FontSize="24" VerticalAlignment="Center"/>
                </StackPanel>
            </NavigationView.Header>

            <!-- Opciones de navegación -->
            <NavigationView.MenuItems>
                <NavigationViewItem Content="Inicio" Icon="Home" Tag="HomePage"/>
                <NavigationViewItem Content="Administrar" Icon="Manage" Tag="AdminPage"/>
                <NavigationViewItem Content="Buscar" Icon="Find" Tag="SearchPage"/>
            </NavigationView.MenuItems>

            <!-- Contenido principal -->
            <Frame x:Name="ContentFrame"/>
        </NavigationView>
    </Grid>
</Page>

```

``` c#
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;

namespace MyApp
{
    public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();

            // Manejar la navegación cuando se selecciona un elemento
            NavView.SelectionChanged += NavView_SelectionChanged;

            // Opcional: Navegar a la página de inicio al cargar la aplicación
            ContentFrame.Navigate(typeof(HomePage));
        }

        private void NavView_SelectionChanged(NavigationView sender, NavigationViewSelectionChangedEventArgs args)
        {
            if (args.IsSettingsSelected)
            {
                // Navegar a la página de configuración si es necesario
                ContentFrame.Navigate(typeof(SettingsPage));
            }
            else
            {
                var selectedItem = args.SelectedItem as NavigationViewItem;
                if (selectedItem != null)
                {
                    var selectedPageTag = selectedItem.Tag.ToString();

                    // Navegar a la página correspondiente
                    switch (selectedPageTag)
                    {
                        case "HomePage":
                            ContentFrame.Navigate(typeof(HomePage));
                            break;
                        case "AdminPage":
                            ContentFrame.Navigate(typeof(AdminPage));
                            break;
                        case "SearchPage":
                            ContentFrame.Navigate(typeof(SearchPage));
                            break;
                    }
                }
            }
        }
    }
}
## Definir datos en formulario

``` c#
using MyApp.Services;
using System;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;

namespace MyApp
{
    public sealed partial class EditProductPage : Page
    {
        private readonly ApiService _apiService;

        public EditProductPage()
        {
            this.InitializeComponent();
            _apiService = new ApiService();
        }

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);

            if (e.Parameter is int productId)
            {
                // Obtener los datos del producto desde la API
                Product product = await _apiService.GetProductByIdAsync(productId);

                // Setear los datos en los campos del formulario
                NameTextBox.Text = product.Name;
                PriceTextBox.Text = product.Price.ToString();
                CategoryComboBox.SelectedItem = product.Category;
                SupplierComboBox.SelectedItem = product.Supplier;
                UrlTextBox.Text = product.Url;
                DescriptionTextBox.Text = product.Description;
            }
        }

        private void OnSaveButtonClick(object sender, Windows.UI.Xaml.RoutedEventArgs e)
        {
            // Lógica para guardar los cambios, si es necesario
        }
    }
}

```
```
