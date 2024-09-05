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
```
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
## Lista de productos
``` xaml
<Page
    x:Class="MyApp.ProductsPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <GridView
            x:Name="ProductsGridView"
            IsItemClickEnabled="True"
            SelectionMode="None"
            Padding="10"
            ItemsSource="{x:Bind Products}"
            ItemClick="OnProductClick">

            <GridView.ItemTemplate>
                <DataTemplate x:DataType="local:Product">
                    <StackPanel Width="200" Margin="10" Background="White" Padding="10" BorderBrush="Gray" BorderThickness="1">
                        <!-- Imagen del producto -->
                        <Image Source="{x:Bind ImageUrl}" Height="150" Stretch="UniformToFill"/>
                        <!-- Nombre del producto -->
                        <TextBlock Text="{x:Bind Name}" FontSize="18" FontWeight="Bold" TextWrapping="Wrap" Margin="0,10,0,0"/>
                        <!-- Precio del producto -->
                        <TextBlock Text="{x:Bind Price, StringFormat='{}{0:C}'}" FontSize="16" Foreground="Green"/>
                    </StackPanel>
                </DataTemplate>
            </GridView.ItemTemplate>
        </GridView>
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
    public sealed partial class ProductsPage : Page
    {
        private readonly ProductService _productService;
        public List<Product> Products { get; set; }

        public ProductsPage()
        {
            this.InitializeComponent();
            _productService = new ProductService();
        }

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);

            // Obtener productos de la API
            Products = await _productService.GetProductsAsync();
            
            // Refrescar la interfaz de usuario
            ProductsGridView.ItemsSource = Products;
        }

        private void OnProductClick(object sender, ItemClickEventArgs e)
        {
            // Lógica para manejar el clic en un producto, por ejemplo, mostrar detalles
            var clickedProduct = e.ClickedItem as Product;
            if (clickedProduct != null)
            {
                // Navegar a una página de detalles del producto, si es necesario
                // Frame.Navigate(typeof(ProductDetailPage), clickedProduct);
            }
        }
    }
}

```
## Slider
``` xaml
<Page
    x:Class="MyApp.SliderPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <FlipView x:Name="SliderFlipView" Width="400" Height="300" HorizontalAlignment="Center" VerticalAlignment="Center">
            <FlipView.ItemTemplate>
                <DataTemplate x:DataType="local:Slide">
                    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                        <Image Source="{x:Bind ImageUrl}" Width="300" Height="200" Stretch="UniformToFill"/>
                        <TextBlock Text="{x:Bind Description}" FontSize="18" FontWeight="Bold" Margin="10,10,10,0" TextAlignment="Center"/>
                    </StackPanel>
                </DataTemplate>
            </FlipView.ItemTemplate>
        </FlipView>
    </Grid>
</Page>

```

Define una clase Slide que represente cada diapositiva con la imagen y el texto asociado.
``` c#
public class Slide
{
    public string ImageUrl { get; set; }
    public string Description { get; set; }
}

```

En el archivo SliderPage.xaml.cs, configura un temporizador para cambiar las diapositivas automáticamente cada 2 segundos.
``` c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;
using Windows.UI.Xaml;

namespace MyApp
{
    public sealed partial class SliderPage : Page
    {
        private List<Slide> Slides;
        private DispatcherTimer timer;
        private int currentSlideIndex = 0;

        public SliderPage()
        {
            this.InitializeComponent();
            InitializeSlides();
            StartSliderTimer();
        }

        private void InitializeSlides()
        {
            // Definir una lista de diapositivas con imágenes y descripciones
            Slides = new List<Slide>
            {
                new Slide { ImageUrl = "ms-appx:///Assets/Imagen1.jpg", Description = "Texto para la primera imagen" },
                new Slide { ImageUrl = "ms-appx:///Assets/Imagen2.jpg", Description = "Texto para la segunda imagen" },
                new Slide { ImageUrl = "ms-appx:///Assets/Imagen3.jpg", Description = "Texto para la tercera imagen" }
            };

            // Asignar las diapositivas al FlipView
            SliderFlipView.ItemsSource = Slides;
        }

        private void StartSliderTimer()
        {
            // Configurar un temporizador para cambiar de diapositiva cada 2 segundos
            timer = new DispatcherTimer();
            timer.Interval = TimeSpan.FromSeconds(2);
            timer.Tick += Timer_Tick;
            timer.Start();
        }

        private void Timer_Tick(object sender, object e)
        {
            // Cambiar a la siguiente diapositiva
            currentSlideIndex = (currentSlideIndex + 1) % Slides.Count;
            SliderFlipView.SelectedIndex = currentSlideIndex;
        }

        protected override void OnNavigatedFrom(NavigationEventArgs e)
        {
            // Detener el temporizador si se navega fuera de la página
            timer.Stop();
            base.OnNavigatedFrom(e);
        }
    }
}

```
Explicación:

    Modelo Slide:
        Este modelo contiene las propiedades ImageUrl y Description para representar cada imagen y su descripción.

    FlipView:
        El control FlipView se utiliza para mostrar las diapositivas en un formato de carrusel. Cada ítem del FlipView es una combinación de una imagen y un texto, organizados en un StackPanel.
        Se usa DataTemplate para enlazar la lista de diapositivas a los controles dentro de cada ítem.

    Temporizador (DispatcherTimer):
        Un DispatcherTimer cambia automáticamente la diapositiva cada 2 segundos, actualizando la propiedad SelectedIndex del FlipView para avanzar al siguiente ítem.
        El índice se actualiza de manera cíclica utilizando el operador módulo (%) para reiniciar el ciclo cuando llega al final de la lista.

    Carga de Diapositivas:
        En el método InitializeSlides, se inicializa una lista de objetos Slide con las imágenes y textos.
        Esta lista se asigna a la propiedad ItemsSource del FlipView.

    Control de Navegación:
        Cuando se navega fuera de la página, el temporizador se detiene para evitar posibles problemas de rendimiento.


## Barra de navegacion
``` xaml
<Page
    x:Class="MyApp.AppShell"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    mc:Ignorable="d">

    <Grid>
        <!-- Barra de navegación -->
        <NavigationView
            x:Name="NavView"
            PaneDisplayMode="Top"
            IsBackButtonVisible="Collapsed"
            Background="LightGray">

            <!-- Contenido de la barra de navegación -->
            <NavigationView.MenuItems>
                <!-- Logotipo / Título -->
                <TextBlock Text="Techstore" FontSize="24" VerticalAlignment="Center" Margin="10" FontWeight="Bold"/>
                
                <!-- Elementos del menú -->
                <NavigationViewItem Content="Inicio" Icon="Home" Tag="HomePage"/>
                <NavigationViewItem Content="Administración" Icon="Manage" Tag="AdminPage"/>
                <NavigationViewItem Content="Buscar" Icon="Find" Tag="SearchPage"/>
            </NavigationView.MenuItems>

            <!-- Frame para cargar las páginas -->
            <Frame x:Name="ContentFrame"/>
        </NavigationView>
    </Grid>
</Page>

```
Explicación:

    NavigationView:
        Se usa para crear una barra de navegación en la parte superior con un modo de visualización en la parte superior (PaneDisplayMode="Top").
        Contiene un TextBlock como título de la tienda ("Techstore") y tres elementos de menú: "Inicio", "Administración" y "Buscar".

    NavigationViewItem:
        Cada uno de los ítems tiene un texto (Content), un ícono (Icon), y una etiqueta (Tag) que servirá para identificar qué página debe cargarse.

    Frame:
        Se define un Frame para que todas las páginas se carguen dentro de este contenedor.


En el archivo AppShell.xaml.cs, necesitas manejar los clics en los elementos del menú y cargar las páginas correspondientes en el Frame.

``` c#
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;

namespace MyApp
{
    public sealed partial class AppShell : Page
    {
        public AppShell()
        {
            this.InitializeComponent();
            // Iniciar con la página de inicio
            ContentFrame.Navigate(typeof(HomePage));
        }

        private void NavView_ItemInvoked(NavigationView sender, NavigationViewItemInvokedEventArgs args)
        {
            if (args.InvokedItemContainer != null)
            {
                // Obtener el Tag del ítem invocado
                var pageTag = args.InvokedItemContainer.Tag.ToString();

                // Navegar según la página seleccionada
                switch (pageTag)
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

```

Explicación:

    NavView_ItemInvoked:
        Este método se ejecuta cuando el usuario selecciona una opción en la barra de navegación.
        Se obtiene el Tag del ítem invocado para identificar a qué página debe navegarse.
        Según el Tag, se navega a la página correspondiente (HomePage, AdminPage, SearchPage).

    ContentFrame.Navigate:
        Usa el Frame para cargar la página seleccionada dentro del área de contenido.

    Página por Defecto:
        La página de inicio (HomePage) se muestra al cargar la aplicación por primera vez.


Para que la barra de navegación esté siempre presente, configura el App.xaml.cs para que la primera página sea AppShell.

``` c#
using Windows.ApplicationModel.Activation;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;

namespace MyApp
{
    sealed partial class App : Application
    {
        public App()
        {
            this.InitializeComponent();
        }

        protected override void OnLaunched(LaunchActivatedEventArgs e)
        {
            Frame rootFrame = Window.Current.Content as Frame;

            if (rootFrame == null)
            {
                // Crear el Frame si aún no existe
                rootFrame = new Frame();
                Window.Current.Content = rootFrame;
            }

            if (rootFrame.Content == null)
            {
                // Mostrar el AppShell como la página principal
                rootFrame.Navigate(typeof(AppShell));
            }

            // Asegurarse de que la ventana esté activa
            Window.Current.Activate();
        }
    }
}

```

