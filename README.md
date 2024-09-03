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
