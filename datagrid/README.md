```xaml
<Page
    x:Class="MyApp.DataGridPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <!-- Contenedor para los filtros -->
        <StackPanel HorizontalAlignment="Stretch" VerticalAlignment="Top" Margin="10">
            <!-- Filtro general -->
            <TextBox x:Name="GeneralFilterBox" PlaceholderText="Buscar por nombre o categoría..." TextChanged="OnGeneralFilterChanged"/>

            <!-- Filtros por columna -->
            <StackPanel Orientation="Horizontal" Margin="0,10,0,10">
                <TextBox x:Name="NameFilterBox" PlaceholderText="Filtrar por nombre" Width="200" TextChanged="OnColumnFilterChanged"/>
                <TextBox x:Name="CategoryFilterBox" PlaceholderText="Filtrar por categoría" Width="200" Margin="10,0,0,0" TextChanged="OnColumnFilterChanged"/>
                <TextBox x:Name="PriceFilterBox" PlaceholderText="Filtrar por precio" Width="200" Margin="10,0,0,0" TextChanged="OnColumnFilterChanged"/>
            </StackPanel>
        </StackPanel>

        <!-- DataGrid / ListView -->
        <ListView x:Name="ProductsListView" ItemsSource="{x:Bind FilteredProducts}" Margin="10,50,10,10">
            <ListView.Header>
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="Nombre" Width="200" FontWeight="Bold"/>
                    <TextBlock Text="Categoría" Width="200" FontWeight="Bold"/>
                    <TextBlock Text="Precio" Width="100" FontWeight="Bold"/>
                </StackPanel>
            </ListView.Header>

            <ListView.ItemTemplate>
                <DataTemplate x:DataType="local:Product">
                    <StackPanel Orientation="Horizontal">
                        <TextBlock Text="{x:Bind Name}" Width="200"/>
                        <TextBlock Text="{x:Bind Category}" Width="200"/>
                        <TextBlock Text="{x:Bind Price, StringFormat='{}{0:C}'}" Width="100"/>
                    </StackPanel>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
    </Grid>
</Page>
```

Explicación:

    Filtros Generales:
        Se coloca un TextBox en la parte superior para realizar una búsqueda general por nombre o categoría.

    Filtros por Columna:
        Tres TextBox adicionales permiten filtrar por columna (nombre, categoría, precio).

    ListView como DataGrid:
        Usamos un ListView con un StackPanel para representar las columnas de cada fila. Los encabezados también están organizados con TextBlock.

    Productos Filtrados:
        La propiedad ItemsSource del ListView está enlazada a una colección filtrada, FilteredProducts, que actualizará dinámicamente su contenido según los filtros aplicados.


Paso 2: Code-Behind para Implementar Filtros

En el archivo DataGridPage.xaml.cs, agregaremos la lógica para aplicar filtros a los productos.
DataGridPage.xaml.cs

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using Windows.UI.Xaml.Controls;

namespace MyApp
{
    public sealed partial class DataGridPage : Page
    {
        // Lista de productos original
        private List<Product> AllProducts;

        // Lista de productos filtrados
        private List<Product> FilteredProducts;

        public DataGridPage()
        {
            this.InitializeComponent();
            InitializeProducts();
            ApplyFilters();
        }

        // Inicializar la lista de productos
        private void InitializeProducts()
        {
            AllProducts = new List<Product>
            {
                new Product { Name = "Laptop", Category = "Electronics", Price = 1200.99 },
                new Product { Name = "Tablet", Category = "Electronics", Price = 499.99 },
                new Product { Name = "Smartphone", Category = "Electronics", Price = 799.99 },
                new Product { Name = "Shoes", Category = "Clothing", Price = 59.99 },
                new Product { Name = "Shirt", Category = "Clothing", Price = 29.99 }
            };

            FilteredProducts = new List<Product>(AllProducts);
        }

        // Evento para aplicar el filtro general
        private void OnGeneralFilterChanged(object sender, TextChangedEventArgs e)
        {
            ApplyFilters();
        }

        // Evento para aplicar los filtros por columna
        private void OnColumnFilterChanged(object sender, TextChangedEventArgs e)
        {
            ApplyFilters();
        }

        // Lógica para aplicar los filtros
        private void ApplyFilters()
        {
            var generalFilter = GeneralFilterBox.Text.ToLower();
            var nameFilter = NameFilterBox.Text.ToLower();
            var categoryFilter = CategoryFilterBox.Text.ToLower();
            var priceFilter = PriceFilterBox.Text.ToLower();

            // Filtrar la lista de productos
            FilteredProducts = AllProducts.Where(p =>
                (string.IsNullOrEmpty(generalFilter) || p.Name.ToLower().Contains(generalFilter) || p.Category.ToLower().Contains(generalFilter)) &&
                (string.IsNullOrEmpty(nameFilter) || p.Name.ToLower().Contains(nameFilter)) &&
                (string.IsNullOrEmpty(categoryFilter) || p.Category.ToLower().Contains(categoryFilter)) &&
                (string.IsNullOrEmpty(priceFilter) || p.Price.ToString().Contains(priceFilter))
            ).ToList();

            // Actualizar la vista
            ProductsListView.ItemsSource = FilteredProducts;
        }
    }

    // Modelo de datos para los productos
    public class Product
    {
        public string Name { get; set; }
        public string Category { get; set; }
        public double Price { get; set; }
    }
}
```

Explicación:

    Modelo Product:
        Define la estructura básica de los productos, con propiedades como Name, Category y Price.

    Inicializar Productos:
        En el método InitializeProducts, se crea una lista de productos de ejemplo. Esta lista representa los datos que aparecerán en el DataGrid.

    Aplicar Filtros:
        El método ApplyFilters toma el texto de los filtros de cada columna y aplica las condiciones usando LINQ para filtrar la lista de productos.
        El filtro general busca coincidencias en el nombre o categoría.
        Los filtros por columna aplican condiciones específicas a cada propiedad (nombre, categoría, o precio).

    Actualizar la Vista:
        Después de aplicar los filtros, el ItemsSource del ListView se actualiza con la lista filtrada, FilteredProducts.

Resultado:

Este código mostrará una lista de productos con un "DataGrid" personalizado usando ListView. Los usuarios podrán filtrar los productos tanto de manera general como por columnas individuales (nombre, categoría, y precio).
Personalización:

    Ordenación por Columna: Puedes agregar funcionalidad adicional para ordenar los productos al hacer clic en los encabezados de las columnas.
    Filtros Más Avanzados: Podrías agregar un rango de precios para filtrar productos dentro de un límite inferior y superior.
    Estilos y Diseño: Puedes mejorar el diseño del ListView agregando bordes, espaciados o aplicando plantillas visuales personalizadas.
