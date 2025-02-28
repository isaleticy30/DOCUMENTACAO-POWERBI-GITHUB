# Objetivo do Relatório
Analisar e ter um melhor gerenciamento de ASOs candidatos e colaboradores (efetivados).  
Obs. O relatório possui 3 páginas: Navegador (Menu de navegação), Temporários e Efetivados

# Fonte de Dados
__Banco de Dados:__ MariaDB  
Views:  
* acompanhamento_asos_2  
* acompanhamento_asos_agendamentos  
* power_bi_pessoas_temporario  
* power_bi_pessoas 

# Agendamento de Atualização
Fuso Horário (UTC-03:00) Brasília  
Frequência das Atualizações: diariamente  
Hora de Atualização:08:30 AM

# Processo de ETL (Extração, Transformação e Carregamento) no Power Query
__acompanhamento_asos_2__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_acompanhamento_asos_2_View = dolpenge_views_Database{[Name="view_acompanhamento_asos_2",Kind="View"]}[Data],  
    #"Tipo Alterado" = Table.TransformColumnTypes(view_acompanhamento_asos_2_View,{{"matricula_Dolp", Int64.Type}})  
in  
    #"Tipo Alterado"

__Explicação dos procedimentos:__  
1. Conexão com a Fonte de Dados  
* O código se conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Acessa o banco de dados chamado dolpenge_views.  
* Obtém os dados da view chamada view_acompanhamento_asos_2, que provavelmente já contém dados processados para análise no Power BI.  

2. Altera o tipo de dado:  
* O código altera o tipo da coluna "matricula_Dolp" para Int64 (inteiro de 64 bits), garantindo que os dados sejam tratados corretamente.  
* A palavra-chave in define o que será retornado no final da consulta, que, nesse caso, é a tabela com a coluna matricula_Dolp convertida para Int64.

__View acompanhamento_asos_agendamentos:__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_acompanhamento_asos_agendamentos_View = dolpenge_views_Database{[Name="view_acompanhamento_asos_agendamentos",Kind="View"]}[Data],  
    #"Tipo Alterado" = Table.TransformColumnTypes(view_acompanhamento_asos_agendamentos_View,{{"cnpj", Int64.Type}, {"data_prevista", type date}, {"data_registro", type date}})  
in  
    #"Tipo Alterado"

__Explicação dos procedimentos:__  
1. Conexão com a Fonte de Dados  
* O código se conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Acessa o banco de dados chamado dolpenge_views.  
* Obtém os dados da view chamada view_acompanhamento_asos_agendamentos_View, que provavelmente já contém dados processados para análise no Power BI.  

2. Alteração dos tipos de dados  
* Altera os tipos de dados das seguintes colunas:  
    * "cnpj" → Int64 (inteiro de 64 bits) → Provavelmente para armazenar CNPJs como números.  
    * "data_prevista" → Date → Define essa coluna como data.  
    * "data_registro" → Date → Define essa coluna como data.  
* A palavra-chave in define o resultado final da consulta, que será a tabela já com os tipos ajustados.

__View power_bi_pessoas_temporario:__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_pessoas_temporario_View = dolpenge_views_Database{[Name="view_power_bi_pessoas_temporario",Kind="View"]}[Data],  
    #"Tipo Alterado" = Table.TransformColumnTypes(view_power_bi_pessoas_temporario_View,{{"cpf", Int64.Type}, {"cnh", Int64.Type},  
    {"matricula_Dolp", Int64.Type}, {"num_cel_1", Int64.Type}}),  
    #"Linhas Filtradas2" = Table.SelectRows(#"Tipo Alterado", each ([funcao_geral] = "ELETRICISTA DE REDES DE DISTRIBUIÇÃO" or [funcao_geral] =  
    "ELETRICISTA LINHA VIVA" or [funcao_geral] =  
    "ELETRICISTA MONTADOR" or [funcao_geral] = "INSTALADOR ELÉTRICO" or [funcao_geral] = "SUPERVISOR")),  
    #"Consultas Mescladas" = Table.NestedJoin(#"Linhas Filtradas2", {"idtb_oper_pessoa"}, ASOs, {"idtb_oper_pessoa"}, "ASOs", JoinKind.LeftOuter),  
    #"ASOs Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas", "ASOs", {"dt_lancamento"}, {"ASOs.dt_lancamento"}),  
    #"Consultas Mescladas1" = Table.NestedJoin(#"ASOs Expandido", {"idtb_oper_pessoa"}, ASO_AGENDADO, {"idtb_oper_pessoa"}, "ASO_AGENDADO", JoinKind.LeftOuter),  
    #"ASO_AGENDADO Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas1", "ASO_AGENDADO", {"data_prevista"}, {"ASO_AGENDADO.data_prevista"}),  
    #"Valor Substituído" = Table.ReplaceValue(#"ASO_AGENDADO Expandido",null,#date(1900, 1, 1),Replacer.ReplaceValue,{"ASO_AGENDADO.data_prevista"}),  
    #"Valor Substituído1" = Table.ReplaceValue(#"Valor Substituído",null,#date(1900, 1, 1),Replacer.ReplaceValue,{"ASOs.dt_lancamento"}),  
    // Criando a coluna de status com base na data  
    #"Status ASO" = Table.AddColumn(#"Valor Substituído1", "Status ASO", each   
        if ([ASO_AGENDADO.data_prevista] = null or [ASO_AGENDADO.data_prevista] = #date(1900, 1, 1)) and  
           ([ASOs.dt_lancamento] = null or [ASOs.dt_lancamento] = #date(1900, 1, 1)) then "Aguardando Agendamento do ASO"  
        else if ([ASO_AGENDADO.data_prevista] <> null and [ASO_AGENDADO.data_prevista] <> #date(1900, 1, 1)) and  
                ([ASOs.dt_lancamento] = null or [ASOs.dt_lancamento] = #date(1900, 1, 1)) then "ASO Agendado"  
        else if ([ASO_AGENDADO.data_prevista] = null or [ASO_AGENDADO.data_prevista] = #date(1900, 1, 1)) and  
                ([ASOs.dt_lancamento] <> null and [ASOs.dt_lancamento] <> #date(1900, 1, 1)) then "ASO Lançado (Não Consta Cadastro do Agendamento do ASO)"  
        else "ASO Lançado"  
    ),  
    #"Tipo Alterado1" = Table.TransformColumnTypes(#"Status ASO",{{"Status ASO", type text}}),  
    #"Linhas Filtradas" = Table.SelectRows(#"Tipo Alterado1", each not List.Contains({"CUIABÁ - MT", "PALMAS - TO", "RIO VERDE - GO"}, [base])),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Linhas Filtradas", each [dt_admissao] >= #date(2024, 12, 2))  
in  
    #"Linhas Filtradas1"

__Explicação dos procedimentos:__  
1. Conexão com a Fonte de Dados  
* O código se conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Acessa o banco de dados chamado dolpenge_views.  
* Obtém os dados da view chamada view_power_bi_pessoas_temporario, que provavelmente já contém dados processados para análise no Power BI.

2. Alteração de Tipos de Dados  
* Converte as colunas cpf, cnh, matricula_Dolp e num_cel_1 para o tipo Int64 (número inteiro de 64 bits).

3. Filtragem por Função  
* Filtra apenas as pessoas que possuem uma das seguintes funções:  
    * ELETRICISTA DE REDES DE DISTRIBUIÇÃO  
    * ELETRICISTA LINHA VIVA  
    * ELETRICISTA MONTADOR  
    * INSTALADOR ELÉTRICO  
    * SUPERVISOR

4. Junção com a Tabela de ASO  
* Faz um JOIN (esquerdo) com a tabela ASOs usando a chave idtb_oper_pessoa.  
* Expande a coluna dt_lancamento, que contém a data de lançamento do ASO.

5. Junção com a Tabela de Agendamento do ASO  
* Faz outro JOIN (esquerdo) com a tabela ASO_AGENDADO usando a chave idtb_oper_pessoa.  
* Expande a coluna data_prevista, que contém a data prevista para o agendamento do ASO.

6. Substituição de Valores Nulos  
* Substitui valores nulos nas colunas ASO_AGENDADO.data_prevista e ASOs.dt_lancamento por 01/01/1900, para facilitar comparações futuras.

7. Criação da Coluna de Status do ASO  
* Cria a coluna "Status ASO" com base nas datas:  
    * "Aguardando Agendamento do ASO" → Nenhuma data preenchida.  
    * "ASO Agendado" → Data de agendamento preenchida, mas sem lançamento.  
    * "ASO Lançado (Não Consta Cadastro do Agendamento do ASO)" → Data de lançamento preenchida, mas sem agendamento.  
    * "ASO Lançado" → Ambas as datas preenchidas.
* Define a coluna "Status ASO" como texto.

8. Filtragem por Base  
* Exclui registros das bases:  
    * CUIABÁ - MT  
    * PALMAS - TO  
    * RIO VERDE - GO

9. Filtragem pela Data de Admissão  
* Mantém apenas funcionários admitidos a partir de 02/12/2024.

__View power_bi_pessoas:__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_pessoas_View = dolpenge_views_Database{[Name="view_power_bi_pessoas",Kind="View"]}[Data],  
    #"Linhas Filtradas" = Table.SelectRows(view_power_bi_pessoas_View, each ([situacao] = "Em Atividade")),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Linhas Filtradas", each [dt_admissao] >= #date(2024, 12, 2)),  
    #"Linhas Filtradas2" = Table.SelectRows(#"Linhas Filtradas1", each ([funcao_geral] = "ELETRICISTA DE LINHA VIVA" or [funcao_geral] =  
    "ELETRICISTA DE REDES DE DISTRIBUIÇÃO" or [funcao_geral] = "ELETRICISTA MONTADOR" or [funcao_geral] = "ENCARREGADO" or [funcao_geral] =  
    "ENCARREGADO ITINERANTE" or [funcao_geral] = "INSTALADOR ELÉTRICO" or [funcao_geral] = "SUPERVISOR") and ([base] <>  
    "CUIABÁ - MT" and [base] <> "PALMAS - TO" and [base] <> "RIO VERDE - GO")),  
    #"Consultas Mescladas" = Table.NestedJoin(#"Linhas Filtradas2", {"idtb_oper_pessoa"}, ASO_AGENDADO, {"idtb_oper_pessoa"},  
    "ASO_AGENDADO", JoinKind.LeftOuter),  
    #"ASO_AGENDADO Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas", "ASO_AGENDADO", {"data_prevista"}, {"ASO_AGENDADO.data_prevista"}),  
    #"Consultas Mescladas1" = Table.NestedJoin(#"ASO_AGENDADO Expandido", {"idtb_oper_pessoa"}, ASOs, {"idtb_oper_pessoa"},  
    "ASOs", JoinKind.LeftOuter),  
    #"ASOs Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas1", "ASOs", {"dt_lancamento"}, {"ASOs.dt_lancamento"}),  
    #"Valor Substituído" = Table.ReplaceValue(#"ASOs Expandido",null,#date(1900, 1, 1),Replacer.ReplaceValue,{"ASO_AGENDADO.data_prevista"}),  
    #"Valor Substituído1" = Table.ReplaceValue(#"Valor Substituído",null,#date(1900, 1, 1),Replacer.ReplaceValue,{"ASOs.dt_lancamento"})  
in  
    #"Valor Substituído1"  

__Explicação dos procedimentos:__  
1. Conexão com a Fonte de Dados  
* O código se conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Acessa o banco de dados chamado dolpenge_views.  
* Obtém os dados da view chamada view_power_bi_pessoas_View, que provavelmente já contém dados processados para análise no Power BI.

2. Filtros
* Mantém apenas funcionários ativos, ou seja, onde a coluna situacao é "Em Atividade"  
* Filtra os funcionários admitidos a partir de 02/12/2024  
* Filtra funcionários que ocupam cargos específicos, como:  
    * ELETRICISTA DE LINHA VIVA  
    * ELETRICISTA DE REDES DE DISTRIBUIÇÃO  
    * ELETRICISTA MONTADOR  
    * ENCARREGADO  
    * ENCARREGADO ITINERANTE  
    * INSTALADOR ELÉTRICO  
    * SUPERVISOR  
* Exclui funcionários das bases:  
    * CUIABÁ - MT  
    * PALMAS - TO  
    * RIO VERDE - GO

3. Junção com Tabelas  
- Juntando com ASO Agendado:  
* Faz um JOIN esquerdo (Left Outer Join) com a tabela ASO_AGENDADO usando a chave idtb_oper_pessoa, para obter informações de agendamento do ASO  
* Expande a coluna data_prevista, que contém a data prevista do ASO

-  Juntando com ASOs Lançados:  
* Faz outro JOIN (esquerdo) com a tabela ASOs usando a chave idtb_oper_pessoa, para obter informações sobre ASOs lançados.  
* Expande a coluna dt_lancamento, que contém a data de lançamento do ASO.

4. Substituição de Valores Nulos
* Substitui valores nulos na coluna ASO_AGENDADO.data_prevista por 01/01/1900  
* Substitui valores nulos na coluna ASOs.dt_lancamento por 01/01/1900  
Obs.: Essa substituição facilita comparações, pois valores nulos podem causar problemas em cálculos e filtros

# Colunas Calculadas e Medidas  
__Tabela Pessoas - Coluna ".DataFormatadaAgendamento1":__  
>.DataFormatadaAgendamento1 =  
IF(  
    'PESSOAS'[ASO_AGENDADO.data_prevista] = DATE(1900, 1, 1),  
    BLANK(),  
    VALUE(DATE(YEAR('PESSOAS'[ASO_AGENDADO.data_prevista]), MONTH('PESSOAS'[ASO_AGENDADO.data_prevista]), DAY('PESSOAS'[ASO_AGENDADO.data_prevista])))  
)

__Explicação do Código__  


__Tabela Pessoas - Coluna ".DataFormatadaLancamento1":__  
>.DataFormatadaLancamento1 =  
IF(  
    'PESSOAS'[ASOs.dt_lancamento] = DATE(1900, 1, 1),  
    BLANK(),  
    VALUE(DATE(YEAR('PESSOAS'[ASOs.dt_lancamento]), MONTH('PESSOAS'[ASOs.dt_lancamento]), DAY('PESSOAS'[ASOs.dt_lancamento])))  
)

__Explicação do Código__  
Tem o objetivo de formatar a data do ASO agendado, tratando casos onde o valor seja 01/01/1900.  
1. Verifica se a data é "1900-01-01"  
    * Se a data prevista do ASO for 01/01/1900, significa que não há um agendamento real (foi substituído no Power Query).  
2. Se for "1900-01-01", retorna um valor em branco (BLANK())  
    * Isso evita exibir uma data falsa no relatório.  
3. Se for uma data válida, converte para um formato numérico  
    * DATE(YEAR(...), MONTH(...), DAY(...)) recria a data extraindo ano, mês e dia da coluna ASO_AGENDADO.data_prevista.  
    * VALUE(...) garante que o valor seja tratado corretamente como um número no Power BI.

__Tabela Pessoas - Coluna ".DataFormatadaSemAgendamento1":__  
>.DataFormatadaSemAgendamento1 =  
IF(  
    TRUNC('PESSOAS'[ASO_AGENDADO.data_prevista]) = DATE(1900, 1, 1),  
    DATE(1900, 1, 1),  
    BLANK()  
)

__Explicação do Código__  
O objetivo é identificar os funcionários sem agendamento de ASO, ou seja, aqueles cuja data prevista foi substituída por 01/01/1900.  
1. Arredonda a data para remover possíveis frações de segundo  
    * A função TRUNC() remove a parte decimal da data (caso existam horas, minutos, segundos).  
    * Isso evita problemas ao comparar a data diretamente com 01/01/1900.  
2. Compara com 01/01/1900  
    * Se a data for exatamente 01/01/1900, significa que não há um ASO agendado (porque essa data foi usada no Power Query para preencher valores nulos)  
3. Se a condição for verdadeira, retorna 01/01/1900  
    * Mantém a data 01/01/1900 para indicar que não há agendamento.  
4. Se a condição for falsa, retorna BLANK()  
    * Se houver uma data real de agendamento, deixa a célula em branco para evitar exibição de informações irrelevantes.

__Tabela Pessoas - Coluna ".DataFormatadaSemLancamento1":__  
>.DataFormatadaSemLancamento1 =  
IF(  
    FORMAT('PESSOAS'[ASOs.dt_lancamento], "dd/MM/yyyy") = "01/01/1900",  
    DATE(1900, 1, 1),  
    BLANK()  
)

__Explicação do Código__  
 Identificar registros onde não há lançamento do ASO (exame médico ocupacional), verificando se a data do lançamento foi substituída por 01/01/1900  
 1. Formata a data como texto (dd/MM/yyyy)  
    * Transforma a data da coluna ASOs.dt_lancamento em um texto no formato "01/01/1900".  
    * Isso permite comparar a data com um valor fixo.  
Obs: O uso de FORMAT() retorna um texto, o que pode afetar cálculos posteriores.  
2. Compara com "01/01/1900"  
    * No Power Query, valores nulos ou ausentes podem ter sido substituídos por 01/01/1900, indicando que o ASO não foi lançado.  
3. Se a data for 01/01/1900, retorna 01/01/1900  
    * Mantém a indicação de que o ASO ainda não foi registrado.  
4. Se houver um ASO lançado, retorna BLANK()  
    * Se a data for diferente de 01/01/1900, deixa a célula em branco.  
    * Isso facilita análises e evita poluir relatórios com valores irrelevantes.

__Tabela Pessoas - Medida ".ASOSQueNãoEstãoAgendadosELançados1":__  
>.ASOSQueNãoEstãoAgendadosELançados1 =  
COALESCE(  
    CALCULATE(  
        DISTINCTCOUNT('PESSOAS'[idtb_oper_pessoa]),  
        FILTER(  
            'PESSOAS',  
            'PESSOAS'[ASO_AGENDADO.data_prevista] = DATE(1900, 1, 1) &&  
            'PESSOAS'[ASOs.dt_lancamento] = DATE(1900, 1, 1)  
        )  
    ),  
    0  
)

__Explicação do Código__  
 Conta a quantidade de funcionários que não têm ASO agendado nem lançado  
 1. Filtra apenas registros sem ASO agendado e sem lançamento  
    * ASO_AGENDADO.data_prevista = 01/01/1900 → Indica que não há um ASO agendado.  
    * ASOs.dt_lancamento = 01/01/1900 → Indica que não há um ASO lançado.  
    * Apenas funcionários que não têm ASO agendado nem registrado são considerados.  
2. Conta os funcionários distintos (DISTINCTCOUNT)  
    * Usa DISTINCTCOUNT() para contar apenas funcionários únicos.  
    * Isso evita que um mesmo funcionário seja contado mais de uma vez caso apareça repetido na base de dados  
3. Usa CALCULATE() para aplicar o filtro e calcular a contagem  
    * CALCULATE() redefine o contexto da medida, aplicando o filtro antes de contar os funcionários.  
4. Se o resultado for BLANK(), retorna 0 com COALESCE()  
    * COALESCE() substitui valores em branco (BLANK()) por 0, garantindo que a medida sempre retorne um número.  
    * Isso evita problemas em gráficos e tabelas do Power BI.

__Tabela Pessoas - Medida ".TotalColaboradores1":__  
>.TotalColaboradores1 =  
DISTINCTCOUNT('PESSOAS'[idtb_oper_pessoa])

__Explicação do Código__  
Conta o número total de funcionários distintos (idtb_oper_pessoa) na tabela 'PESSOAS'.  
1. DISTINCTCOUNT('PESSOAS'[idtb_oper_pessoa])  
    * Conta apenas valores únicos da coluna idtb_oper_pessoa.  
    * Se um funcionário aparece mais de uma vez na tabela, ele será contado apenas uma vez.  
    * Isso evita duplicações que poderiam inflar os números.

__Tabela Pessoas Temporário - Coluna ".DataFormatadaAgendamento":__  
>.DataFormatadaAgendamento =  
IF(  
    'PESSOAS_TEMPORARIO'[ASO_AGENDADO.data_prevista] = DATE(1900, 1, 1),  
    BLANK(),  
    VALUE(DATE(YEAR('PESSOAS_TEMPORARIO'[ASO_AGENDADO.data_prevista]), MONTH('PESSOAS_TEMPORARIO'[ASO_AGENDADO.data_prevista]),  
    DAY('PESSOAS_TEMPORARIO'[ASO_AGENDADO.data_prevista])))  
)

__Explicação do Código__  
Formata a data do ASO agendado, garantindo que valores inválidos (01/01/1900) sejam substituídos por BLANK() (vazio)  
1. Verifica se a data do ASO agendado é inválida (01/01/1900)  
    * Se a data for 01/01/1900, retorna BLANK() (vazio).  
    * Isso evita que datas inválidas apareçam em gráficos e tabelas no Power BI.  
2. Se a data for válida, converte para um formato correto  
    * Usa YEAR(), MONTH() e DAY() para extrair os componentes da data.  
    * DATE(YEAR(...), MONTH(...), DAY(...)) recria a data corretamente.  
    * VALUE(...) garante que o resultado seja um valor numérico, útil para cálculos.

__Tabela Pessoas Temporário - Coluna ".DataFormatadaLancamento":__  
>.DataFormatadaLancamento =  
IF(  
    'PESSOAS_TEMPORARIO'[ASOs.dt_lancamento] = DATE(1900, 1, 1),  
    BLANK(),  
    VALUE(DATE(YEAR('PESSOAS_TEMPORARIO'[ASOs.dt_lancamento]), MONTH('PESSOAS_TEMPORARIO'[ASOs.dt_lancamento]),  
    DAY('PESSOAS_TEMPORARIO'[ASOs.dt_lancamento])))  
)

__Explicação do Código__  
Formata a data de lançamento do ASO, substituindo valores inválidos (01/01/1900) por BLANK() (vazio) e mantendo as datas válidas no formato correto.  
1. Verifica se a data de lançamento do ASO é inválida (01/01/1900)  
    * Se a data for 01/01/1900, retorna BLANK() (vazio), evitando exibições incorretas no Power BI.  
2. Se a data for válida, recria o formato correto  
    * Usa YEAR(), MONTH(), e DAY() para extrair os componentes da data.  
    * DATE(YEAR(...), MONTH(...), DAY(...)) recria a data corretamente.  
    * VALUE(...) garante que o resultado seja um valor numérico.

__Tabela Pessoas Temporário - Coluna ".DataFormatadaSemAgendamento":__  
>.DataFormatadaSemAgendamento =  
IF(  
    TRUNC('PESSOAS_TEMPORARIO'[ASO_AGENDADO.data_prevista]) = DATE(1900, 1, 1),  
    DATE(1900, 1, 1),  
    BLANK()  
)

__Explicação do Código__  
Essa medida identifica registros sem um agendamento válido do ASO.  
Se a data de agendamento do ASO for 01/01/1900, a medida retorna essa mesma data. Caso contrário, retorna BLANK() (vazio).  
1. Verifica se a data de agendamento do ASO é inválida (01/01/1900)  
    * TRUNC(...) garante que a comparação funcione corretamente caso a coluna tenha valores de data e hora.  
    * Se a data for 01/01/1900, significa que o ASO ainda não foi agendado.  
2. Retorna 01/01/1900 para registros sem agendamento e BLANK() para os demais  
    * Se a data for 01/01/1900, mantém esse valor como saída.  
    * Se a data for diferente, retorna BLANK().

__Tabela Pessoas Temporário - Coluna ".DataFormatadaSemLancamento":__  
>.DataFormatadaSemLancamento =  
IF(  
    FORMAT('PESSOAS_TEMPORARIO'[ASOs.dt_lancamento], "dd/MM/yyyy") = "01/01/1900",  
    DATE(1900, 1, 1),  
    BLANK()  
)

__Explicação do Código__  
Identifica registros onde o lançamento do ASO não foi realizado (caso a data de lançamento seja 01/01/1900). Se isso acontecer,  
ela retorna a data 1900-01-01; caso contrário, retorna BLANK() (vazio).  
1. Verifica se a data de lançamento do ASO é inválida (01/01/1900)  
    * O FORMAT(..., "dd/MM/yyyy") converte a data para o formato de string dd/MM/yyyy, facilitando a comparação.  
    * Se a data for 01/01/1900, significa que o lançamento do ASO não foi realizado.  
2. Retorna a data 01/01/1900 quando não há lançamento do ASO e BLANK() caso contrário  
    * Se a condição for verdadeira (ou seja, se a data for 01/01/1900), a medida retorna essa data.  
    * Caso contrário, retorna BLANK() (vazio), sinalizando que o lançamento foi feito ou não é necessário.

__Tabela Pessoas Temporário - Medida ".ASOSQueNãoEstãoAgendadosELançados":__  
>.ASOSQueNãoEstãoAgendadosELançados =  
COALESCE(  
    CALCULATE(  
        DISTINCTCOUNT('PESSOAS_TEMPORARIO'[idtb_oper_pessoa]),  
        FILTER(  
            'PESSOAS_TEMPORARIO',  
            'PESSOAS_TEMPORARIO'[ASO_AGENDADO.data_prevista] = DATE(1900, 1, 1) &&  
            'PESSOAS_TEMPORARIO'[ASOs.dt_lancamento] = DATE(1900, 1, 1)  
        )  
    ),  
    0  
)

__Explicação do Código__  
Calcula quantos colaboradores ainda não têm o ASO agendado nem lançado. Ou seja, ela conta o número de pessoas que possuem a data de agendamento do ASO (ASO_AGENDADO.  data_prevista) e a data de lançamento do ASO (ASOs.dt_lancamento) como 01/01/1900, indicando que essas datas ainda não foram definidas.  
Se não houver nenhum colaborador que se enquadre nesse critério, a medida retorna 0 (usando COALESCE).  
1. CALCULATE e DISTINCTCOUNT  
    * Conta o número único de colaboradores (idtb_oper_pessoa) que atendem ao critério especificado no FILTER.  
2. FILTER para Selecionar Pessoas Sem Agendamento e Lançamento  
    * Filtra os colaboradores onde a data de agendamento do ASO e a data de lançamento do ASO são 01/01/1900, indicando que esses eventos não foram realizados  
3. Uso de COALESCE para Garantir que Retorne 0 Quando Não Houver Registros  
    * O COALESCE garante que, se o CALCULATE não retornar nenhum valor (por exemplo, se não houver registros que atendam à condição), o resultado será 0 em vez de BLANK().

__Tabela Pessoas Temporário - Medida ".TotalColaboradores":__  
>.TotalColaboradores =  
DISTINCTCOUNT('PESSOAS_TEMPORARIO'[idtb_oper_pessoa])

__Explicação do Código__  
Conta o número total de funcionários distintos (idtb_oper_pessoa) na tabela 'PESSOAS'.  
1. DISTINCTCOUNT('PESSOAS'[idtb_oper_pessoa])  
    * Conta apenas valores únicos da coluna idtb_oper_pessoa.  
    * Se um funcionário aparece mais de uma vez na tabela, ele será contado apenas uma vez.  
    * Isso evita duplicações que poderiam inflar os números.

# Indicadores KPIs  
O relatório é composto por 2 gráficos (1 grafico de tabela e 1 grafico de mapa), 3 segmentadores de dados e 4 cartões contendo 4 imagens com ações de clique para segmentação de dados em cada aba do relatório.

__Gráfico de Tabela:__  
1. KPI: Agendamento e lançamento do ASO por candidatos temporários e colaboradores efetivados.

__Gráfico de Mapa:__  
1. KPI: A loclaização são as unidades os candidatos e colaboradores efetivados estão.
Obs.: O tamanho da bolha são a quantidade distinta de colaboradore ou candidatos.

__Cartões__  
1. KPI: Total de candidatos e efetivados  
2. KPI: Candidatos e efetivados sem ASO  
3. KPI: Candidatos e efetivados com ASO  
4. KPI: Candidatos e efetivados com ASO agendado

__Segmentadores de Dados (Filtros dinâmicos)__  
1. Função 
2. Data de Admissão  
3. Base