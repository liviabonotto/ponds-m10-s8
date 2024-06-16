# Ponderada Programação M10 S8
## Coletando métricas do ASP.NET Core usando Prometheus e Grafana.

O tutorial completo pode ser encontrado [aqui](https://learn.microsoft.com/pt-br/aspnet/core/log-mon/metrics/metrics?view=aspnetcore-8.0).
As imagens foram subidas no Cloudinary e anexadas neste documento. 

---


### Criar o Aplicativo Inicial

#### 1. Crie um novo projeto ASP.NET Core executando os comandos:

```bash
dotnet new web -o WebMetric
cd WebMetric
dotnet add package OpenTelemetry.Exporter.Prometheus.AspNetCore --prerelease
dotnet add package OpenTelemetry.Extensions.Hosting
```

![1](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/1_rdc30a.png)

---

#### 2. Substitua o conteúdo de `Program.cs` pelo código abaixo:

```bash
using OpenTelemetry.Metrics;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddOpenTelemetry()
    .WithMetrics(builder =>
    {
        builder.AddPrometheusExporter();

        builder.AddMeter("Microsoft.AspNetCore.Hosting",
                         "Microsoft.AspNetCore.Server.Kestrel");
        builder.AddView("http.server.request.duration",
            new ExplicitBucketHistogramConfiguration
            {
                Boundaries = new double[] { 0, 0.005, 0.01, 0.025, 0.05,
                       0.075, 0.1, 0.25, 0.5, 0.75, 1, 2.5, 5, 7.5, 10 }
            });
    });
var app = builder.Build();

app.MapPrometheusScrapingEndpoint();

app.MapGet("/", () => "Hello OpenTelemetry! ticks:"
                     + DateTime.Now.Ticks.ToString()[^3..]);

app.Run();
```

![2](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/2_ou2eap.png)

---

### Exibir Métricas com dotnet-counters

#### 3. Instale a ferramenta `dotnet-counters` se ainda não estiver instalada.

```bash
dotnet tool update -g dotnet-counters
```

#### 4. Monitore as métricas enquanto o aplicativo estiver em execução (lembre de executar o projeto com ```dotnet run```).

```bash
dotnet-counters monitor -n WebMetric --counters Microsoft.AspNetCore.Hosting
```

![3-4](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/4_mffzkg.png)

---

### Exibir métricas do aplicativo de exemplo
Acrescente /metrics à URL em que o projeto está rodando, para exibir o ponto de extremidade de métricas. O navegador exibe as métricas que estão sendo coletadas:

![5](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/5_drfp6p.png)

---

### Configurar o Prometheus

#### 6. Instale o [Prometheus](https://prometheus.io/docs/visualization/grafana/#creating-a-prometheus-graph) seguindo os passos especificados.

![6](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/6_wbjngr.png)

#### 7. Extraia os arquivos

![7](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/7_hsaxvz.png)

#### 8. Inicie o servidor Prometheus executando o arquivo ```Prometheus.exe```.

![8](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/8_vacxzp.png)

Acessando http://localhost:9090/metrics
![9](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/9_yydvvh.png)

#### 9. Substitua o conteúdo do arquivo `prometheus.yml` por:

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: 'MyASPNETApp'
    scrape_interval: 5s # Poll every 5 seconds for a more responsive demo.
    static_configs:
      - targets: ["localhost:5045"]  ## Enter the HTTP port number of the demo app.
```
**Não esqueça de substituir no target pela porta em que sua aplicação está rodando.**

![9-10](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/10_wuyjqi.png)

**Reinicie o Prometheus após substituir o conteúdo no arquivo.**

#### 10. Acesse http://localhost:9090/targets e confirme se o OpenTelemetryTest está no estado UP.

![11](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/11_fgsybd.png)

#### 11. Acesse http://localhost:9090/graph e selecione o ícone ```Abrir gerenciador de métricas``` para ver as métricas disponíveis:

![12](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/12_d99ol5.png)

#### 12. Insira a categoria do contador, como kestrel, na caixa de pequisa, para ver as métricas disponíveis:

![13](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/13_nrmkmz.png)

---

### Mostrar métricas em um painel do Grafana

#### 13. Faça o Download do [Grafana](https://prometheus.io/docs/visualization/grafana/#creating-a-prometheus-graph) seguindo os passos indicados. 

![14](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578018/14_vjt3wc.png)

#### 14. Por padrão, o Grafana estará rodando em http://localhost:3000. O login padrão é "admin"/"admin".

![15](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/15_oufzzz.png)

#### 15. Faça o Download de um template de Dashboard [aqui](https://grafana.com/orgs/dotnetteam).

![16](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/16_lhopkm.png)

![17](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/17_dmkuzx.png)

#### 16. Configure um novo Dashboard usando o template baixado. 

1. Clique na “roda dentada” na barra lateral para abrir o menu Configuração.
2. Clique em “Fontes de dados”.
3. Clique em “Adicionar fonte de dados”.
4. Selecione "Prometheus" como tipo.
5. Defina o URL do servidor Prometheus apropriado (por exemplo, http://localhost:9090/)
6. Ajuste outras configurações de fonte de dados conforme desejado (por exemplo, escolhendo o método de acesso correto).
7. Clique em “Salvar e testar” para salvar a nova fonte de dados.
8. Importe o modelo fazendo upload do arquivo JSON.

![18](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/18_ut5whu.png)

#### 17. O Dashboard está pronto e as métricas já podem ser visualizadas em tempo real. 

![19](https://res.cloudinary.com/dwxvglcja/image/upload/v1718578017/19_wueb1y.png)
