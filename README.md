# Testes de Performance – API de Pets (JMeter)

Projeto de testes de performance com **Apache JMeter 5.6.3** para a [API de Pets](https://github.com/tiagotaq/api-de-pets).

## Arquivos

| Arquivo | Descrição |
| --- | --- |
| `pets-performance-test.jmx` | Plano de testes do JMeter |
| `pets-performance-test.zip` | O mesmo `.jmx` compactado (para entrega) |

## O que o plano de testes faz

Os dois Thread Groups fazem **login antes** de consultar os endpoints:

1. `POST /login` → extrai o `token` JWT da resposta (JSON Extractor `$.token`)
2. `POST /pets` → cria um pet com nome único (`Pet-...-${__UUID()}`) enviando `Authorization: Bearer ${token}`; valida status **201** e campo `id`
3. `GET /pets` → lista os pets com o mesmo token; valida status **200** e que a lista não está vazia

### Thread Group – Modelo Fechado
- Elemento: `Thread Group`
- **10 usuários** fixos, ramp-up de 10 s, **20 iterações** cada, think time de 500 ms
- O login fica num `Once Only Controller`: cada usuário faz login uma vez e reaproveita o token nas iterações
- No modelo fechado, um usuário só começa uma nova iteração depois de terminar a anterior — a carga depende do tempo de resposta do sistema

### Thread Group – Modelo Aberto
- Elemento: `Open Model Thread Group` (nativo do JMeter 5.5+)
- Agenda: `rate(0/s) random_arrivals(10 s) rate(5/s) random_arrivals(40 s) rate(5/s) random_arrivals(10 s) rate(0/s)`
  - sobe de 0 a 5 chegadas/s em 10 s, mantém 5 chegadas/s por 40 s e desce para 0 em 10 s
- Cada chegada é um usuário novo que faz login, `POST /pets` e `GET /pets`
- No modelo aberto, novos usuários chegam numa taxa fixa, independentemente de os anteriores terem terminado

Os Thread Groups rodam em sequência (fechado e depois aberto).

## Como executar

1. Suba a API de Pets:
   ```bash
   git clone https://github.com/tiagotaq/api-de-pets.git
   cd api-de-pets
   npm install
   npm start   # http://localhost:3000
   ```
2. Execute o JMeter em modo não-GUI:
   ```bash
   jmeter -n -t pets-performance-test.jmx -l resultados.jtl -e -o relatorio
   ```
   O relatório HTML fica em `relatorio/index.html`.

### Parâmetros (opcionais)

| Propriedade | Padrão | Exemplo |
| --- | --- | --- |
| `host` | `localhost` | `-Jhost=minha-api.com` |
| `port` | `3000` | `-Jport=8080` |
| `protocol` | `http` | `-Jprotocol=https` |
| `username` | `tiago` | `-Jusername=ricardo` |
| `password` | `123456` | `-Jpassword=456789` |
