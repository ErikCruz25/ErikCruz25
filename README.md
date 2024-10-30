<Window x:Class="MathSolverApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Math Solver App" Height="350" Width="525">
    <Grid>
        <TextBlock Text="Ingrese la ecuación:" Margin="10,10,0,0" VerticalAlignment="Top" HorizontalAlignment="Left" />
        
        <!-- Caja de texto para ingresar la ecuación -->
        <TextBox Name="EquationInput" Width="400" Height="30" Margin="10,40,0,0" VerticalAlignment="Top" HorizontalAlignment="Left" />

        <!-- Botón para resolver la ecuación -->
        <Button Content="Resolver" Width="100" Height="30" Margin="420,40,10,0" VerticalAlignment="Top" HorizontalAlignment="Right" Click="SolveButton_Click" />

        <!-- TextBox para mostrar el resultado -->
        <TextBox Name="ResultOutput" Margin="10,80,10,10" VerticalAlignment="Top" TextWrapping="Wrap" VerticalScrollBarVisibility="Auto" HorizontalScrollBarVisibility="Auto" IsReadOnly="True" />
    </Grid>
</Window>
using System;
using System.Net.Http;
using System.Threading.Tasks;
using System.Windows;
using Newtonsoft.Json.Linq;

namespace MathSolverApp
{
    public partial class MainWindow : Window
    {
        // Coloca tu clave de API de Wolfram Alpha aquí
        private static readonly string WolframAppId = "TU_CLAVE_API_DE_WOLFRAM";

        public MainWindow()
        {
            InitializeComponent();
        }

        // Evento de clic para el botón "Resolver"
        private async void SolveButton_Click(object sender, RoutedEventArgs e)
        {
            string equation = EquationInput.Text;

            if (string.IsNullOrWhiteSpace(equation))
            {
                MessageBox.Show("Por favor, ingrese una ecuación.");
                return;
            }

            try
            {
                // Llamada a la función para resolver la ecuación
                string result = await ResolveEquation(equation);

                // Muestra el resultado en el TextBox ResultOutput
                ResultOutput.Text = result;
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Ocurrió un error: {ex.Message}");
            }
        }

        /// <summary>
        /// Realiza una llamada a la API de Wolfram Alpha para resolver la ecuación
        /// </summary>
        /// <param name="equation">La ecuación a resolver</param>
        /// <returns>Resultado en texto con los pasos de resolución</returns>
        public static async Task<string> ResolveEquation(string equation)
        {
            using (var client = new HttpClient())
            {
                // Construcción del URL para la solicitud a la API de Wolfram Alpha
                var url = $"http://api.wolframalpha.com/v2/query?input={Uri.EscapeDataString(equation)}&format=plaintext&output=JSON&appid={WolframAppId}";
                
                // Realiza la solicitud HTTP a la API
                var response = await client.GetStringAsync(url);

                // Procesa el JSON de la respuesta
                JObject jsonResponse = JObject.Parse(response);

                // Extrae el resultado principal y los pasos si están disponibles
                var pods = jsonResponse["queryresult"]?["pods"];
                if (pods == null) return "No se pudo resolver la ecuación.";

                string resultText = "";

                foreach (var pod in pods)
                {
                    var title = pod["title"]?.ToString();
                    var subpod = pod["subpods"]?.First?["plaintext"]?.ToString();

                    // Agrega el título y el contenido del subpod al resultado final
                    if (!string.IsNullOrEmpty(subpod))
                    {
                        resultText += $"{title}:\n{subpod}\n\n";
                    }
                }

                return resultText;
            }
        }
    }
}
