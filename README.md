# Boleto-Autenticator

# Estrutura do Projeto:
Frontend (HTML + JavaScript)

Tela para inserir os dados do boleto.

Envio dos dados para a API em C#.

Backend (C# - ASP.NET Core)

API que recebe os dados, valida o boleto e retorna a resposta.


# 💡 Passo 1: Criar a Interface em HTML (Frontend)

<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Autenticador de Boletos</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 2rem; }
        input, button { margin-top: 0.5rem; padding: 0.5rem; }
        #resultado { margin-top: 1rem; font-weight: bold; }
    </style>
    <script>
        async function autenticarBoleto(event) {
            event.preventDefault(); // Evita recarregar a página

            const boletoCodigo = document.getElementById('boletoCodigo').value.trim();
            const resultado = document.getElementById('resultado');

            if (!boletoCodigo) {
                resultado.innerText = '⚠️ Por favor, insira o código do boleto.';
                return;
            }

            resultado.innerText = '🔄 Autenticando...';

            try {
                const response = await fetch('https://localhost:5001/api/boletos/autenticar', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ codigo: boletoCodigo })
                });

                if (!response.ok) {
                    throw new Error(`Erro HTTP: ${response.status}`);
                }

                const data = await response.json();
                resultado.innerText = data.autenticado
                    ? '✅ Boleto autenticado com sucesso!'
                    : '❌ Falha na autenticação do boleto!';
            } catch (error) {
                resultado.innerText = `❌ Erro ao autenticar: ${error.message}`;
            }
        }
    </script>
</head>
<body>
    <h1>🔐 Serviço de Autenticação de Boletos</h1>

    <form onsubmit="autenticarBoleto(event)">
        <label for="boletoCodigo">Código do Boleto:</label><br>
        <input type="text" id="boletoCodigo" placeholder="Digite o código do boleto" required><br>
        <button type="submit">Autenticar</button>
    </form>

    <p id="resultado"></p>
</body>
</html>


# 📁 Passo 2: Criar o Modelo
No diretório Models, crie um arquivo chamado BoletoRequest.cs:

namespace BoletoApi.Models
{
    public class BoletoRequest
    {
        public string Codigo { get; set; }
    }
}


# 📁 Passo 3: Criar o Backend em C# (ASP.NET Core)
Primeiro, crie uma aplicação ASP.NET Core no Visual Studio ou use o .NET CLI:

bash
dotnet new webapi -n BoletoAuthenticator

# Dentro da aplicação, crie um controlador para autenticação de boletos.

Controllers/BoletoController.cs
using Microsoft.AspNetCore.Mvc;

namespace BoletoAuthenticator.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class BoletosController : ControllerBase
    {
        // Endpoint para autenticação do boleto
        [HttpPost("autenticar")]
        public IActionResult AutenticarBoleto([FromBody] BoletoRequest request)
        {
            // Aqui você deve implementar a lógica de autenticação do boleto
            var autenticado = ValidarBoleto(request.Codigo);

            return Ok(new { autenticado });
        }

        private bool ValidarBoleto(string codigo)
        {
            // Lógica de validação do boleto (simulação)
            // Em um sistema real, você consultaria um banco de dados ou API externa aqui

            // Exemplo: validar se o código tem exatamente 10 caracteres
            return codigo.Length == 10;
        }
    }

    public class BoletoRequest
    {
        public string Codigo { get; set; }
    }
}


# 📁 Passo 4: Configuração de CORS
Você pode precisar configurar o CORS para permitir que o frontend se comunique com a API.
No arquivo Startup.cs ou Program.cs (depende da versão do .NET), adicione a configuração para o CORS.
No Program.cs:

var builder = WebApplication.CreateBuilder(args);

// Configuração do CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

var app = builder.Build();

// Usar o CORS
app.UseCors("AllowAll");

app.UseAuthorization();

app.MapControllers();

app.Run();


# 🚀 Executar a API No terminal:
dotnet run


# 🌐 Testar com seu HTML
Certifique-se de que o HTML está sendo servido por um servidor local (por exemplo, usando a extensão Live Server do VS Code). 
O navegador pode bloquear requisições fetch se você abrir o HTML diretamente como file://.

# ✅ Teste Rápido com curl (opcional)
curl -X POST https://localhost:5001/api/boletos/autenticar \
-H "Content-Type: application/json" \
-d "{\"codigo\":\"123456789\"}" -k

# Use -k para ignorar o certificado SSL autoassinado.
