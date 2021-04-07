#Para desenvolvimento foi utilizado Visual Studio Code com framework DotNet5.0, Sistema Operacional Linux (Distro Debian Buster) e MySql 8.0.23

#Para instalar o framework DotNet Core para Linux acesse:
https://docs.microsoft.com/pt-br/dotnet/core/install/linux

#Para utilizar C# no Visual Studio Code adicione a extensão "C# for Visual Studio Code (powered by OmniSharp)."

# Para informações referente aos comandos "dotnet" a seguir acesse a documentação disponível em: 
# https://docs.microsoft.com/pt-br/aspnet/core/tutorials/first-mvc-app/start-mvc?view=aspnetcore-5.0&tabs=visual-studio

#Criando e abrindo o projeto no Visual Studio Code

dotnet new mvc -o SalesWebMvc
code -r SalesWebMvc
dotnet dev-certs https --trust

#Para rodar a aplicação:
dotnet run

#Para rodar a aplicação com refresh automático quando allterar o código:
dotnet watch run

#Para compilar o projeto e ver se está com algum erro
dotnet build

#############################################################################################

#Criar dentro da pasta "Model" o arquivo "Department.cs":

namespace SalesWebMvc.Models
{
    public class Department
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}



#############################################################################################


#Instalar gerador de código e adicionar pacotes:

dotnet tool install --global dotnet-ef
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.Extensions.Logging.Debug



#############################################################################################

# Criar o arquivo "SalesWebMvcContext.cs" no diretório "Data":
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using SalesWebMvc.Models;

namespace SalesWebMvc.Data
{
    public class SalesWebMvcContext : DbContext
    {
        public SalesWebMvcContext (DbContextOptions<SalesWebMvcContext> options)
            : base(options)
        {
        }

        public DbSet<SalesWebMvc.Models.Department> Department { get; set; }
    }
}


#############################################################################################

#Instalar plugin de conexão do MySql:
dotnet add package Pomelo.EntityFrameworkCore.MySql --version 5.0.0-alpha.2

#Fonte: https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql

#############################################################################################

# Adicionar imports no arquivo "Startup.cs":
using SalesWebMvc.Data;
using Microsoft.EntityFrameworkCore;
using Pomelo.EntityFrameworkCore.MySql.Infrastructure;

#Alterar o método "ConfigureServices" do arquivo "Startup.cs":

public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();

     services.AddDbContext<SalesWebMvcContext>(options =>
            options.UseMySql(Configuration.GetConnectionString("SalesWebMvcContext"),
            new MySqlServerVersion(new Version(8, 0, 23)),
             builder => builder.MigrationsAssembly("SalesWebMvc")));
}




#############################################################################################

#Instale o MySql de acordo com o seu Sistema Operacional:

https://dev.mysql.com/downloads/


#Opcional: Instalar extensão do MySql no Visual Code para utilizar no lugar do Workbench:
# Procure por "MySQl" desenvolvida por "Jun Han" nas extensões do Visual Studio Code

#No terminal execute o MySql com root:

mysql -uroot -p
#Digite a senha e pressione enter

#Atenção: Os comandos do sql devem ser executados com ponto-e-vírgula no final

#Crie um usuário com acesso ao localhost:
# Onde tem 'user' digite o seu nome mantendo as aspas simples
# Onde está 'strong_password' informe uma senha forte mantendo as aspas simples
# Onde está GRANT ALL PRIVILEGES você pode liberar apenas os privilégios que deseja ao invés de todos

CREATE USER 'user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'strong_password';
GRANT USAGE ON *.* TO 'user'@'localhost';
ALTER USER 'user'@'localhost' REQUIRE NONE WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;
GRANT ALL PRIVILEGES ON `user`.* TO 'user'@'localhost';
FLUSH PRIVILEGES;


#Para verificar os usuários:
select Host, User from mysql.user;

#Feche a conexão do banco do usuário root:
exit;

#Acesse com o usuário novo:

mysql -u novoUsuario -p
#Digite a senha e pressione enter

# Se aparecer mensagem de que não tem permissão de acesso, pode ser que a senha criada é fraca.
# Nesse caso acesse com o usuário root novamente e alter a senha com: ALTER USER 'novoUsuario'@'localhost' IDENTIFIED WITH mysql_native_password BY 'novaSenhaMaisForte';

#############################################################################################


#Alterar arquivo "appsettings.json" com as configurações de conexão do banco:
#ATENÇÃO: Onde está SeuUsuario informe o seu usuário do banco MySql
#ATENÇÃO: Onde está SuaSenha informe o seu usuário do banco MySql


{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "SalesWebMvcContext": "Server=localhost;userid=SeuUsuario;password=SuaSenha;database=saleswebmvcappbd"
  }
}

#############################################################################################


#Executar o comando a seguir dentro da pasta raíz do projeto, onde fica o "Program.cs":
export PATH=$HOME/.dotnet/tools:$PATH

#Gera o controlador a partir do modelos
dotnet aspnet-codegenerator controller -name DepartmentsController -m Department -dc SalesWebMvcContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries


#############################################################################################

#Se não tiver instalado no início, instale a ferramenta de migração
dotnet tool install --global dotnet-ef

#Gere a primeira migração (Cria as tabelas do banco de dados a partir dos modelos):
dotnet ef migrations add InitialCreate

#Se precisar desfazer utilize o comando: dotnet ef migrations remove

#Atualiza o banco de dados:
dotnet ef database update


#############################################################################################


#Compile o projeto e verifique se existe algum erro:
dotnet build

#Execute o projeto:
dotnet run

#Ou execute com: dotnet watch run

#Acesse: https://localhost:5001/Departments/

#Clique em "Create New" e realize testes com as opções de Adcionar, Editar e Deletar

#Verifique no MySql se as informações adicionadas foram gravadas no banco:

#Faça login com o seu usuário e então rode (Pode ser utilizada extensão do MySql no próprio Visual Studio Code):
use saleswebmvcappbd;
select * from Department;

