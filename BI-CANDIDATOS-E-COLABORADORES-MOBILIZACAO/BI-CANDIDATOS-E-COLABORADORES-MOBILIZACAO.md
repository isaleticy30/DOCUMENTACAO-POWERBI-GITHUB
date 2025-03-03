# Objetivo do Relatório
Analisar e gerenciar o processo de contratações (candidatos e efetivados) semanal e retornos da Bernhoeft.  
Obs. O relatório possui 4 páginas: Navegador (Menu de navegação), Candidatos, Bernhoeft e Efetivados.

# Fonte de Dados
__Banco de Dados:__ MariaDB  
Views:  
* view_power_bi_treinamentos  
* view_power_bi_pessoas  
* view_power_bi_pessoas_temporario  
* view_power_bi_treina_participantes

__Planilha WEB:__  
Caminho "https://dolpengenhariacombr-my.sharepoint.com/personal/administrativo03_dolpengenharia_com_br/Documents/DASHBOARDS/MOBILIZA%C3%87%C3%83O%20%C3%82NCORA%20-%20GO.xlsx"  
Obs.: As planilhas usadas são:  
    * Mobilização  
    * Equat Integração

# Agendamento de Atualização
Fuso Horário (UTC-03:00) Brasília  
Frequência das Atualizações: semanal (Segunda a Sexta)  
Horários de Atualização:
08:30 AM
02:00 PM
04:00 PM

# Processo de ETL (Extração, Transformação e Carregamento) no Power Query  
# Fonte de Dados: Maria DB (Views)
__view_power_bi_treinamentos__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_treinamentos_View = dolpenge_views_Database{[Name="view_power_bi_treinamentos",Kind="View"]}[Data],  
    #"Colunas Removidas" = Table.RemoveColumns(view_power_bi_treinamentos_View,{"cargaHoraria"}),  
    #"Dividir Coluna pelas Posições" = Table.SplitColumn(Table.TransformColumnTypes(#"Colunas Removidas", {{"dataInicio", type text}}, "pt-BR"), "dataInicio", Splitter  
    SplitTextByPositions({0, 11}), {"dataInicio.1", "dataInicio.2"}),  
    #"Dividir Coluna pelas Posições1" = Table.SplitColumn(Table.TransformColumnTypes(#"Dividir Coluna pelas Posições", {{"dataFim", type text}}, "pt-BR"), "dataFim"  
    Splitter.SplitTextByPositions({0, 11}), {"dataFim.1", "dataFim.2"}),  
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Dividir Coluna pelas Posições1",{{"dataInicio.1", type date}, {"dataInicio.2", type time},  
    {"dataFim.1", type date}, {"dataFim.2", type time}, {"n_part", Int64.Type}, {"total_sec", Int64.Type}}),  
    #"Colunas Renomeadas" = Table.RenameColumns(#"Tipo Alterado",{{"dataInicio.1", "dataInicio"}, {"dataFim.1", "dataFim"},  
    {"dataInicio.2", "horaInicio"}, {"dataFim.2", "horaFim"}, {"carga_horaria", "totalHoras"}}),  
    #"Personalização Adicionada" = Table.AddColumn(#"Colunas Renomeadas", "CargaHorária", each let  
    duration = if [horaFim] < [horaInicio]  
               then [horaFim] - [horaInicio] + #duration(1, 0, 0, 0)  
               else [horaFim] - [horaInicio],  
    totalMinutes = Duration.TotalMinutes(duration),  
    totalHours = Number.IntegerDivide(totalMinutes, 60),  
    remainingMinutes = Number.Mod(totalMinutes, 60)  
in  
    Text.PadStart(Text.From(totalHours), 2, "0") & ":" & Text.PadStart(Text.From(Number.RoundDown(remainingMinutes)), 2, "0")),  
    #"Tipo Alterado1" = Table.TransformColumnTypes(#"Personalização Adicionada",{{"CargaHorária", type time}}),  
    #"Erros Removidos" = Table.RemoveRowsWithErrors(#"Tipo Alterado1", {"CargaHorária"})  
in  
    #"Erros Removidos"

__Explicação dos procedimentos:__  
1. Conectar ao Banco de Dados MariaDB  
* Conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Não há credenciais explícitas (possivelmente são fornecidas externamente).  
* Acessa o banco de dados "dolpenge_views" dentro do servidor.  
* Obtém os dados da view view_power_bi_treinamentos.

2. Remover Coluna Indesejada
* Exclui a coluna cargaHoraria, pois possivelmente será recalculada.

3. Separar Data e Hora das Colunas dataInicio e dataFim  
* Converte a coluna dataInicio para texto.  
* Divide a string em duas partes:  
    * dataInicio.1 (a data, primeiros 11 caracteres).  
    * dataInicio.2 (a hora, o restante).  
* O mesmo é feito para dataFim:

4. Alteração de Tipos de Dados  
* Converte dataInicio.1 e dataFim.1 para tipo data.  
* Converte dataInicio.2 e dataFim.2 para tipo hora.  
* Define n_part e total_sec como inteiros.

5. Renomear Colunas  
* Ajusta os nomes das colunas para um formato mais claro.

6. Calcular a CargaHorária  
* Calcula a diferença entre horaFim e horaInicio para obter a carga horária.  
* Se horaFim for menor que horaInicio, assume que a atividade passou da meia-noite e adiciona 1 dia (#duration(1, 0, 0, 0)).  
* Converte a duração para minutos.  
* Separa em horas e minutos.  
* Formata para "hh:mm".

7. Ajuste de Tipo da CargaHorária  
* Converte a coluna CargaHorária para tipo time.

8. Remove Erros  
* Remove linhas onde CargaHorária contém erros (possivelmente por dados inconsistentes).  
* Define #"Erros Removidos" como a tabela final retornada.

__view_power_bi_pessoas__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_pessoas_View = dolpenge_views_Database{[Name="view_power_bi_pessoas",Kind="View"]}[Data],  
    #"Colunas Removidas" = Table.RemoveColumns(view_power_bi_pessoas_View,{"dt_nascimento", "escolaridade", "idade", "dt_demissao", "cnh",  
    "cnh_validade", "matricula_Dolp", "matricula_Enel", "num_cel_1", "situacao", "departamento", "cipa", "pcd", "tipo_deficiencia",  
    "tipo", "empresa", "biometria", "login", "id_superior", "superior_imediato", "tipo_funcao"}),  
    #"Colunas Renomeadas" = Table.RenameColumns(#"Colunas Removidas",{{"idtb_oper_pessoa", "ID_Colaborador"}, {"nome", "Colaborador"},  
    {"cpf", "CPF"}, {"status", "Status"}, {"funcao_geral", "Função"}, {"sexo", "Sexo"}, {"dt_admissao", "Dt_Admissao"}, {"base", "Base"}}),  
    #"Linhas Filtradas" = Table.SelectRows(#"Colunas Renomeadas", each ([Status] = "ATIVO")),  
    // Definindo a data base (primeira segunda-feira de Dezembro de 2024)  
    DataBase = #date(2024, 12, 2),  
    // Função para calcular a semana correta  
    calcularSemana = (dataAdmissao as date) =>  
    let  
        // Ajuste para a segunda-feira da semana de admissão  
        DataInicioSemana = Date.AddDays(dataAdmissao, -Date.DayOfWeek(dataAdmissao, Day.Monday)),  
        // Diferença em dias entre a data de admissão e a data base  
        DiferencaEmDias = Duration.Days(DataInicioSemana - DataBase),  
        // Cálculo da semana, arredondado  
        Semana = Number.RoundDown(DiferencaEmDias / 7) + 1  
    in  
        Semana,  
    // Aplicando a função de semana na tabela  
    #"Semana Admissão" = Table.AddColumn(#"Linhas Filtradas", "Semana Admissão", each calcularSemana([Dt_Admissao]), Int64.Type),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Semana Admissão", each ([Função] = "ELETRICISTA DE LINHA VIVA" or [Função] = "ENCARREGADO"  
    or [Função] = "ENCARREGADO ITINERANTE" or [Função] = "ENCARREGADO LINHA VIVA" or [Função] = "ENGENHEIRO ELETRICISTA" or [Função] =  
    "INSTALADOR ELÉTRICO" or [Função] = "SUPERVISOR" or [Função] = "TÉCNICO DE SEGURANÇA DO TRABALHO" or [Função] =  
    "TÉCNICO ELETROTÉCNICO") and ([Base] <> "CUIABÁ - MT" and [Base] <> "PALMAS - TO" and [Base] <> "RIO VERDE - GO"  
    and [Base] <> "VÁRZEA GRANDE - MT")),  
    #"Linhas Filtradas2" = Table.SelectRows(#"Linhas Filtradas1", each [Dt_Admissao] >= #date(2024, 12, 2)),  
    #"Linhas Filtradas3" = Table.SelectRows(#"Linhas Filtradas2", each ([ancora] = "SIM")),  
    #"Colunas Removidas1" = Table.RemoveColumns(#"Linhas Filtradas3",{"Função"})  
in  
    #"Colunas Removidas1"

__Explicação dos procedimentos:__  
1. Conectar ao Banco de Dados MariaDB  
* Conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Não há credenciais explícitas (possivelmente são fornecidas externamente).  
* Acessa o banco de dados "dolpenge_views" dentro do servidor.  
* Obtém os dados da view view_power_bi_pessoas.

2. Remove Colunas Desnecessárias  
* Exclui diversas colunas irrelevantes, deixando apenas as essenciais.

3. Renomeia Colunas  
* Renomeia colunas para melhor compreensão.

4. Filtros
* Mantém apenas os colaboradores com status "ATIVO".  
* Define uma data base para cálculos, que é a primeira segunda-feira de dezembro de 2024

5. Cria Função Para Calcular a Semana da Admissão  
* Ajusta a data da admissão para a segunda-feira daquela semana.  
* Calcula a diferença em dias entre essa data ajustada e a data base.  
* Determina a semana relativa (quantas semanas desde a data base).

6. Aplica Função na Tabela  
* Adiciona uma nova coluna "Semana Admissão" com o resultado do cálculo.

7. Filtrar Apenas Determinadas Funções  
* Mantém apenas colaboradores com funções específicas.

8. Exclui Algumas Bases  
* Remove colaboradores que pertencem às bases Cuiabá, Palmas, Rio Verde e Várzea Grande.

9. Filtros 2 
* Mantém apenas colaboradores admitidos a partir de 2 de dezembro de 2024.  
* Seleciona apenas aqueles cujo campo "âncora" está marcado como "SIM".

10. Remove a Coluna Função
* Exclui a coluna Função, pois já foi usada para filtragem.

11. Retorna o Resultado Final  
* Define a tabela final como saída do script.

__view_power_bi_pessoas__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_pessoas_temporario_View = dolpenge_views_Database{[Name="view_power_bi_pessoas_temporario",Kind="View"]}[Data],  
    #"Colunas Renomeadas" = Table.RenameColumns(view_power_bi_pessoas_temporario_View,{{"idtb_empresa_reg", "ID_Base"},  
    {"idtb_oper_pessoa", "ID_Colaborador"}, {"nome", "Colaborador"}, {"cpf", "CPF"}, {"sexo", "Sexo"}, {"dt_nascimento",  
    "Dt_Nascimento"}, {"dt_admissao", "Dt_Admissao"}, {"funcao_geral", "Função"}, {"escolaridade", "Escolaridade"},  
    {"base", "Base"}, {"status", "Status"}, {"idade", "Idade"}, {"dt_demissao", "Dt_Demissao"}, {"cnh", "CNH"}, {"cnh_validade",  
    "CNH_Validade"}, {"matricula_Dolp", "Matricula_Dolp"}, {"matricula_Enel", "Matricula_Enel"}, {"situacao", "Situacao"},  
    {"departamento", "Departamento"}, {"cipa", "Cipa"}, {"pcd", "PCD"}, {"tipo_deficiencia", "Tipo_Deficiencia"}, {"tipo", "Tipo"},  
    {"empresa", "Empresa"}, {"tipo_funcao", "Tipo_Funcao"}, {"biometria", "Biometria"}, {"login", "Login"}, {"id_superior", "ID_Superior"},  
    {"superior_imediato", "Superior_Imediato"}}),  
    #"Linhas Filtradas" = Table.SelectRows(#"Colunas Renomeadas", each ([Base] <> "CUIABÁ - MT" and [Base] <> "PALMAS - TO" and [Base] <> "RIO VERDE - GO")  
    and ([Função] = "INSTALADOR ELÉTRICO" or [Função] = "SUPERVISOR")),  
    // Definindo a data base (primeira segunda-feira de Dezembro de 2024)  
    DataBase = #date(2024, 12, 2),  
    // Função para calcular a semana correta  
    calcularSemana = (dataAdmissao as date) =>  
    let  
        // Ajuste para a segunda-feira da semana de admissão  
        DataInicioSemana = Date.AddDays(dataAdmissao, -Date.DayOfWeek(dataAdmissao, Day.Monday)),  
        // Diferença em dias entre a data de admissão e a data base  
        DiferencaEmDias = Duration.Days(DataInicioSemana - DataBase),  
        // Cálculo da semana, arredondado  
        Semana = Number.RoundDown(DiferencaEmDias / 7) + 1  
    in  
        Semana,  
    // Aplicando a função de semana na tabela  
    #"Semana Admissão" = Table.AddColumn(#"Linhas Filtradas", "Semana Admissão", each calcularSemana([Dt_Admissao]), Int64.Type),  
    #"Valor Substituído" = Table.ReplaceValue(#"Semana Admissão",null,#date(1900, 1, 1),Replacer.ReplaceValue,{"Dt_Admissao"}),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Valor Substituído", each ([Dt_Admissao] <> #date(1900, 1, 1)))  
in  
    #"Linhas Filtradas1"

__Explicação dos procedimentos:__  
1. Conectar ao Banco de Dados MariaDB  
* Conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Não há credenciais explícitas (possivelmente são fornecidas externamente).  
* Acessa o banco de dados "dolpenge_views" dentro do servidor.  
* Obtém os dados da view view_power_bi_pessoas.

2. Renomear Colunas  
* Renomeia colunas para facilitar a leitura.

3. Filtros  
* Exclui colaboradores das bases:
    * CUIABÁ - MT
    * PALMAS - TO
    * RIO VERDE - GO
* Mantém apenas funções específicas:
    * INSTALADOR ELÉTRICO
    * SUPERVISOR

4. Definir a Data Base  
* Define a primeira segunda-feira de dezembro de 2024 como referência.

5. Criar Função para Calcular a Semana de Admissão  
* Ajusta a data de admissão para a segunda-feira daquela semana.  
* Calcula a diferença em dias entre essa data ajustada e a data base.  
* Determina a semana relativa.

6. Adicionar Coluna "Semana Admissão"  
* Aplica a função calcularSemana e cria a coluna "Semana Admissão".

7. Substituir Valores Nulos na Data de Admissão  
* Substitui valores nulos na coluna "Dt_Admissao" por 01/01/1900 (valor fictício).

8. Remover Registros com Data de Admissão Inválida  
* Remove as linhas que possuem a data fictícia 01/01/1900.

9. Retornar o Resultado Final  
* Define a tabela final como saída do script.

__view_power_bi_treina_participantes__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_treina_participantes_View = dolpenge_views_Database{[Name="view_power_bi_treina_participantes",Kind="View"]}[Data],  
    #"Consultas Mescladas" = Table.NestedJoin(view_power_bi_treina_participantes_View, {"participante"}, PESSOAS, {"Colaborador"}, "PESSOAS", JoinKind.LeftOuter),  
    #"PESSOAS Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas", "PESSOAS", {"Dt_Admissao"}, {"PESSOAS.Dt_Admissao"}),  
    #"Linhas Filtradas" = Table.SelectRows(#"PESSOAS Expandido", each [PESSOAS.Dt_Admissao] >= #date(2024, 12, 2)),  
    #"Duplicatas Removidas" = Table.Distinct(#"Linhas Filtradas", {"idLista"}),  
    #"Consultas Mescladas1" = Table.NestedJoin(#"Duplicatas Removidas", {"idLista"}, TREINAMENTOS, {"idForm50"}, "TREINAMENTOS", JoinKind.LeftOuter),  
    #"TREINAMENTOS Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas1", "TREINAMENTOS", {"id_instrutor", "instrutor", "dataInicio",
    "horaInicio", "dataFim", "horaFim", "id_certificado", "CargaHorária"}, {"TREINAMENTOS.id_instrutor", "TREINAMENTOS.instrutor",  
    "TREINAMENTOS.dataInicio", "TREINAMENTOS.horaInicio", "TREINAMENTOS.dataFim", "TREINAMENTOS.horaFim", "TREINAMENTOS.id_certificado",  
    "TREINAMENTOS.CargaHorária"})  
in  
    #"TREINAMENTOS Expandido"

__Explicação dos procedimentos:__  
1. Conectar ao Banco de Dados MariaDB  
* Conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Não há credenciais explícitas (possivelmente são fornecidas externamente).  
* Acessa o banco de dados "dolpenge_views" dentro do servidor.  
* Obtém os dados da view view_power_bi_treina_participantes.

2. Mesclar com a Tabela de Pessoas  
* Realiza um JOIN (junção) externo à esquerda (LeftOuter) entre:  
    * Tabela de participantes (view_power_bi_treina_participantes_View) ➝ Campo "participante".  
    * Tabela de colaboradores (PESSOAS) ➝ Campo "Colaborador".  
* Isso adiciona os dados da tabela PESSOAS à de participantes, mantendo todos os participantes, mesmo que não tenham correspondência na tabela de pessoas.

3. Expandir a Data de Admissão  
* Expande a coluna "Dt_Admissao" da tabela PESSOAS, trazendo a data de admissão dos colaboradores para a tabela de participantes.

4. Filtrar Apenas Participantes Admitidos Após 02/12/2024  
* Mantém apenas participantes cuja data de admissão (PESSOAS.Dt_Admissao) seja igual ou posterior a 02/12/2024.

5. Remover Duplicatas Baseadas no ID da Lista  
* Remove registros duplicados com base no campo "idLista".

6. Mesclar com a Tabela de Treinamentos  
* Realiza um JOIN externo à esquerda (LeftOuter) entre:  
    * Tabela de participantes filtrados (Duplicatas Removidas) ➝ Campo "idLista".  
    * Tabela de treinamentos (TREINAMENTOS) ➝ Campo "idForm50".  
* Isso adiciona informações dos treinamentos para cada participan

7. Expandir Detalhes dos Treinamentos  
* Expande as seguintes colunas da tabela TREINAMENTOS:  
    * "id_instrutor" ➝ ID do instrutor.  
    * "instrutor" ➝ Nome do instrutor.  
    * "dataInicio" / "horaInicio" ➝ Data e hora de início do treinamento.  
    * "dataFim" / "horaFim" ➝ Data e hora de término.  
    * "id_certificado" ➝ ID do certificado gerado.  
    * "CargaHorária" ➝ Carga horária do treinamento.

8. Retornar a Tabela Final  
* Define a tabela resultante como saída do script.

# Fonte de Dados: Planilha WEB
__Tabela MOBILIZACAO__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = Excel.Workbook(Web.Contents("https://dolpengenhariacombr-my.sharepoint.com/personal/administrativo03_dolpengenharia_com_br/Documents/DASHBOARDS
    MOBILIZA%C3%87%C3%83O%20%C3%82NCORA%20-%20GO.xlsx"), null, true),  
    MOBILIZAÇÃO_Sheet = Fonte{[Item="MOBILIZAÇÃO",Kind="Sheet"]}[Data],  
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(MOBILIZAÇÃO_Sheet, [PromoteAllScalars=true]),  
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",{{"DATA_MOBILIZAÇÃO", type date}, {"ENVIO", type date},  
    {"STATUS DO REGISTRO", type text}, {"ADMISSÃO", type date}, {"COLABORADOR", type text}, {"FUNÇÃO", type text}, {"BASE", type text},  
    {"ID_Colaborador", Int64.Type}, {"DATA DO ASO", type date}, {"ASO", type text}, {"NR 10 BÁSICO (INICIAL)", type text},  
    {"NR 10 BÁSICO (RECICLADO)", type text}, {"NR10 SEP (INICIAL)", type text}, {"NR10 SEP (RECICLADO)", type text}, {"NR 35 (INICIAL)", type text},  
    {"NR 35 (RECICLADO)", type text}, {"OS", type text}, {"AUT SEP", type text}, {"AUT NRS", type text}, {"CTPS", type text}, {"EPI", type text},  
    {"FICHA DE REGISTRO", type text}, {"RG/CPF", type text}, {"CERT ELETRICISTA", type text}, {"INTEGRAÇÃO", type text}, {"PRIMEIROS SOCORROS", type text},  
    {"CNH", type text}, {"QUALIFICAÇÃO TÉCNICA", type text}}),  
    #"Personalização Adicionada" = Table.AddColumn(#"Tipo Alterado", "SemanaAdmissão", each let  
    // Cria uma lista de datas com as colunas "DATA DO ASO" e "ADMISSÃO"  
    TodasDatas = { [DATA DO ASO], [ADMISSÃO] },  
    // Ordena as datas de forma crescente  
    Ordenadas = List.Sort(TodasDatas, Order.Ascending),  
    // Determina a "SemanaAdmissão" com base na posição das datas ordenadas  
    SemanaAdmissao = List.PositionOf(Ordenadas, List.Min(Ordenadas), Occurrence.First) + 1  
in  
    SemanaAdmissao),  
    #"Tipo Alterado1" = Table.TransformColumnTypes(#"Personalização Adicionada",{{"SemanaAdmissão", Int64.Type}}),  
    #"Colunas Renomeadas" = Table.RenameColumns(#"Tipo Alterado1",{{"SemanaAdmissão", "Semana Admissão"}}),  
    #"Linhas Filtradas" = Table.SelectRows(#"Colunas Renomeadas", each ([FUNÇÃO] <> null))  
in  
    #"Linhas Filtradas"

__Explicação dos procedimentos:__  
1. Conexão ao arquivo Excel   
* Web.Contents("https://dolpengenhariacombr-my.sharepoint.com/personal/administrativo03_dolpengenharia_com_br/Documents/DASHBOARDS
    MOBILIZA%C3%87%C3%83O%20%C3%82NCORA%20-%20GO.xlsx"): URL válida para carregar o arquivo.  
* MOBILIZAÇÃO_Sheet: Obtém os dados da aba chamada "MOBILIZAÇÃO".

2. Promove Cabeçalhos  
Define a primeira linha da planilha como os nomes das colunas.

3. Altera Tipos de Dados  
* Converte os dados para os tipos corretos:  
    * Datas (type date) → DATA_MOBILIZAÇÃO, ADMISSÃO, DATA DO ASO, etc.  
    * Texto (type text) → COLABORADOR, FUNÇÃO, NR10 SEP, OS, etc.  
    * Números inteiros (Int64.Type) → ID_Colaborado

4. Cria a Coluna "Semana Admissão"  
* Cria uma lista (TodasDatas) com as datas "DATA DO ASO" e "ADMISSÃO".  
* Ordena as datas da menor para a maior (List.Sort()).  
* Identifica a posição da menor data e soma 1 (List.PositionOf()).  
* Converte a coluna "SemanaAdmissão" para número inteiro.  
* Altera o nome da coluna para "Semana Admissão".

5. Filtrar Apenas Linhas com Função Não Vazia  
* Remove registros onde a coluna "FUNÇÃO" esteja vazia (null).

6.  Retorna a Tabela Final  
* Define a tabela final como saída do script.

__Tabela EQUAT_INTEGRAÇÃO__  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = Excel.Workbook(Web.Contents("https://dolpengenhariacombr-my.sharepoint.com/personal/administrativo03_dolpengenharia_com_br/Documents/DASHBOARDS/MOBILIZA%C3%87%C3%83O%20%C3%82NCORA%20-%20GO.xlsx"), null, true),  
    #"Linhas alternadas removidas1" = Table.AlternateRows(Fonte,3,2,6),  
    #"Linhas alternadas removidas" = Table.AlternateRows(#"Linhas alternadas removidas1",3,5,3),  
    #"EQUAT INTEGRAÇÃO_Sheet" = #"Linhas alternadas removidas"{[Item="EQUAT.INTEGRAÇÃO",Kind="Sheet"]}[Data],  
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(#"EQUAT INTEGRAÇÃO_Sheet", [PromoteAllScalars=true]),  
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",{{"NOME", type text}, {"ID_Colaborador", Int64.Type},  
    {"ADMISSÃO", type date}, {"FUNÇÃO", type text}, {"LOCAL DE SERVIÇO", type text}, {"DATA DE ENVIO", type text},  
    {"STATUS_BERNHOEFT", type text}, {"DATA DA INTEGRAÇÃO", type date}, {"DATA DE APROVAÇÃO", type date}, {"INTEGRAÇÃO", type text},  
    {"OBSERVAÇÃO", type text}, {"CRACHÁ EQUATORIAL", type text}}),  
    #"Linhas Filtradas" = Table.SelectRows(#"Tipo Alterado", each [NOME] <> null and [NOME] <> ""),  
    #"Consultas Mescladas" = Table.NestedJoin(#"Linhas Filtradas", {"NOME"}, MOBILIZACAO, {"COLABORADOR"}, "MOBILIZACAO", JoinKind.LeftOuter),  
    #"MOBILIZACAO Expandido" = Table.ExpandTableColumn(#"Consultas Mescladas", "MOBILIZACAO", {"Semana Admissão"}, {"MOBILIZACAO.Semana Admissão"}),  
    #"Colunas Renomeadas1" = Table.RenameColumns(#"MOBILIZACAO Expandido",{{"MOBILIZACAO.Semana Admissão", "MOBILIZACAO.SemanaAdmissão"}}),  
    #"Linhas Filtradas1" = Table.SelectRows(#"Colunas Renomeadas1", each ([FUNÇÃO] <> "")),  
    #"Tipo Alterado1" = Table.TransformColumnTypes(#"Linhas Filtradas1",{{"DATA DE ENVIO", type date}}),  
    #"Linhas Filtradas2" = Table.SelectRows(#"Tipo Alterado1", each ([STATUS_BERNHOEFT] <> "DESLIGADO"))  
in  
    #"Linhas Filtradas2"

__Explicação dos procedimentos:__  
1. Conexão ao arquivo Excel   
* Web.Contents("https://dolpengenhariacombr-my.sharepoint.com/personal/administrativo03_dolpengenharia_com_br/Documents/DASHBOARDS
    MOBILIZA%C3%87%C3%83O%20%C3%82NCORA%20-%20GO.xlsx"): URL válida para carregar o arquivo.  
* MOBILIZAÇÃO_Sheet: Obtém os dados da aba chamada "MOBILIZAÇÃO".

2. Remove Linhas Alternadas  
* Remove linhas alternadas:  
    * A primeira operação remove 2 linhas a cada 6, começando da terceira linha.  
    * A segunda operação remove 5 linhas a cada 3, começando da terceira linha.

3. Seleciona a Aba "EQUAT.INTEGRAÇÃO"  
* Obtém os dados da aba "EQUAT.INTEGRAÇÃO".

4. Promove Cabeçalhos  
* Define a primeira linha da planilha como os nomes das colunas.

5. Altera os Tipos de Dados  
* Converte colunas para os tipos apropriados:  
    * Datas (type date) → ADMISSÃO, DATA DE ENVIO, DATA DA INTEGRAÇÃO, etc.  
    * Texto (type text) → NOME, FUNÇÃO, STATUS_BERNHOEFT, etc.  
    * Números inteiros (Int64.Type) → ID_Colaborador.

6. Remover Linhas com "NOME" Vazio  
* Remove linhas onde NOME seja nulo ou vazio.

7.  Juntar Dados com a Tabela "MOBILIZACAO"  
* Faz uma junção (LEFT JOIN) entre:  
    * Tabela atual (EQUAT.INTEGRAÇÃO)  
    * Tabela "MOBILIZACAO", baseada nos campos:  
        * "NOME" (na tabela atual)  
        * "COLABORADOR" (na tabela MOBILIZACAO).

8. Expandir a Coluna "Semana Admissão"  
* Adiciona a coluna "Semana Admissão" da tabela MOBILIZACAO.

9. Renomear a Coluna  
* Muda o nome da coluna "MOBILIZACAO.Semana Admissão" para "MOBILIZACAO.SemanaAdmissão"

10. Remover Linhas com "FUNÇÃO" Vazia  
* Exclui registros onde a coluna "FUNÇÃO" esteja vazia.

11. Corrigir Tipo da Coluna "DATA DE ENVIO"  
* Converte "DATA DE ENVIO" para tipo data.

12. Filtrar Somente Funcionários Ativos  
* Remove funcionários desligados, ou seja, exclui linhas onde "STATUS_BERNHOEFT" = "DESLIGADO".

13. Retornar a Tabela Final  
* Define a tabela final como saída do script.

# Tabelas Criadas
__Tabela criada no PowerQuery:__ .Treinamentos_Participantes_ID  
Código M dos procedimentos no Power Query:  
>let  
    Fonte = MariaDB.Contents("sgddolp.com", null, null),  
    dolpenge_views_Database = Fonte{[Name="dolpenge_views",Kind="Database"]}[Data],  
    view_power_bi_treina_participantes_View = dolpenge_views_Database{[Name="view_power_bi_treina_participantes",Kind="View"]}[Data],  
    #"Colunas Removidas" = Table.RemoveColumns(view_power_bi_treina_participantes_View,{"idParticipante", "participante", "assinado"}),  
    #"Duplicatas Removidas" = Table.Distinct(#"Colunas Removidas")  
in  
    #"Duplicatas Removidas"

__Explicação dos procedimentos:__  
1. Conectar ao Banco de Dados MariaDB  
* Conecta ao banco de dados MariaDB no servidor "sgddolp.com".  
* Não há credenciais explícitas (possivelmente são fornecidas externamente).  
* Acessa o banco de dados "dolpenge_views" dentro do servidor.  
* Obtém os dados da view view_power_bi_treina_participantes.

2. Remover Colunas Não Necessárias
* Exclui as colunas:
    * "idParticipante" → Pode ser um identificador numérico.
    * "participante" → Nome ou outro dado do participante.
    * "assinado" → Indica se o treinamento foi assinado.
Motivo: As colunas foram consideradas irrelevantes para a análise.

3. Remove Duplicatas  
* Remove registros duplicados, garantindo que a tabela final contenha apenas linhas únicas.

4. Retorna a Tabela Final
* Define o resultado final da consulta, que será carregado no Power BI.

__Tabela criada com DAX:__ .Pessoas_TreinamentosPart  
Código DAX dos procedimentos:  
>.Pessoas_TreinamentosPart = 
UNION(
    SELECTCOLUMNS(
        FILTER(
            'PESSOAS',
            NOT 'PESSOAS'[Colaborador] IN DISTINCT('TREINAMENTOS_PARTICIPANTES'[participante])
        ),
        "Colaborador", 'PESSOAS'[Colaborador],
        "Status", "Pessoas",
        "Assinatura", BLANK(),
        "Dt_Admissão", 'PESSOAS'[Dt_Admissao],
        "Carga Horária", BLANK(),
        "dataFim", BLANK(),
        "dataInicio", BLANK(),
        "horaFim", BLANK(),
        "horaInicio", BLANK(),
        "id_Certificado", BLANK(),
        "id_Instrutor", BLANK(),
        "Instrutor", BLANK(),
        "id_Lista", BLANK()
    ),
    SELECTCOLUMNS(
        'TREINAMENTOS_PARTICIPANTES',
        "Colaborador", 'TREINAMENTOS_PARTICIPANTES'[participante],
        "Status", "Treinamentos Participantes",
        "Assinatura", 'TREINAMENTOS_PARTICIPANTES'[assinado],
        "Dt_Admissão", 'TREINAMENTOS_PARTICIPANTES'[PESSOAS.Dt_Admissao],
        "Carga Horária", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.CargaHorária],
        "dataFim", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.dataFim],
        "dataInicio", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.dataInicio],
        "horaFim", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.horaFim],
        "horaInicio", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.horaInicio],
        "id_Certificado", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.id_certificado],
        "id_Instrutor", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.id_instrutor],
        "Instrutor", 'TREINAMENTOS_PARTICIPANTES'[TREINAMENTOS.instrutor],
        "id_Lista", 'TREINAMENTOS_PARTICIPANTES'[idLista]
    )

__Explicação dos procedimentos:__  
Essa tabela combina informações sobre colaboradores e treinamentos em que participaram, diferenciando aqueles que participaram de treinamentos dos que não participaram.