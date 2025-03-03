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
Código DAX:  
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

__Explicação do Código:__  
Essa tabela combina informações sobre colaboradores e treinamentos em que participaram, diferenciando aqueles que participaram de treinamentos dos que não participaram.  
1. Colaboradores que não participaram de treinamentos  
* Usa FILTER para encontrar registros na tabela 'PESSOAS' cujo [Colaborador] não aparece na tabela 'TREINAMENTOS_PARTICIPANTES'.  
* SELECTCOLUMNS extrai as colunas desejadas, preenchendo com BLANK() os campos relacionados a treinamentos (como carga horária, datas e assinaturas).

2. Colaboradores que participaram de treinamentos  
* Usa SELECTCOLUMNS para trazer diretamente os dados da tabela 'TREINAMENTOS_PARTICIPANTES', incluindo detalhes dos treinamentos em que participaram.

Objetivo do Código:  
Criar uma tabela que consolida todos os colaboradores, separando aqueles que participaram de treinamentos dos que não participaram.  
* Quem não participou aparece com "Status" = "Pessoas" e sem informações de treinamento.  
* Quem participou aparece com "Status" = "Treinamentos Participantes" e os detalhes do treinamento.

__Tabela criada com DAX:__ .Temporarios_Efetivos
Código DAX:  
>.Temporarios_Efetivos =  
VAR DataBase = DATE(2024, 12, 2) -- Primeira segunda-feira de Dezembro de 2024  
RETURN  
UNION(  
    SELECTCOLUMNS(  
        FILTER(  
            'PESSOAS',  
            'PESSOAS'[Dt_Admissao] >= DATE(2024, 12, 1) -- Inclui todas as datas a partir de 1º de Dezembro de 2024  
        ),  
        "Base", 'PESSOAS'[Base],  
        "ID_Colaborador", 'PESSOAS'[ID_Colaborador],  
        "Colaborador", 'PESSOAS'[Colaborador],  
        "Função", 'PESSOAS'[funcao_especifica],  
        "Sexo", 'PESSOAS'[Sexo],  
        "Dt_Admissao", 'PESSOAS'[Dt_Admissao],  
        "Semana Admissão",  
            INT(  
                DIVIDE(  
                    DATEDIFF(  
                        DataBase,  
                        'PESSOAS'[Dt_Admissao],  
                        DAY  
                    ),  
                    7  
                )  
            ) + 1,  
        "Status", "Efetivado"  
    ),  
    SELECTCOLUMNS(  
        FILTER(  
            'CANDIDATOS_TEMPORARIO',  
            'CANDIDATOS_TEMPORARIO'[Dt_Admissao] >= DATE(2024, 12, 1) -- Inclui todas as datas a partir de 1º de Dezembro de 2024  
        ),  
        "Base", 'CANDIDATOS_TEMPORARIO'[Base],  
        "ID_Colaborador", 'CANDIDATOS_TEMPORARIO'[ID_Colaborador],  
        "Colaborador", 'CANDIDATOS_TEMPORARIO'[Colaborador],  
        "Função", 'CANDIDATOS_TEMPORARIO'[Função],  
        "Sexo", 'CANDIDATOS_TEMPORARIO'[Sexo],  
        "Dt_Admissao", 'CANDIDATOS_TEMPORARIO'[Dt_Admissao],  
        "Semana Admissão",  
            INT(  
                DIVIDE(  
                    DATEDIFF(  
                        DataBase,  
                        'CANDIDATOS_TEMPORARIO'[Dt_Admissao],  
                        DAY  
                    ),  
                    7  
                )  
            ) + 1,  
        "Status", "Temporário"  
    )  
)

__Explicação do Código:__  
A tabela resulta da união (UNION) de duas seleções (SELECTCOLUMNS):

* Efetivados (pessoas contratadas como efetivas a partir de 1º de dezembro de 2024).  
* Temporários (pessoas contratadas como temporárias a partir da mesma data).

A variável DataBase armazena a data 02/12/2024 (primeira segunda-feira de dezembro), usada para calcular em qual semana de admissão cada pessoa foi contratada.

1. A coluna "Semana Admissão" determina quantas semanas se passaram entre DataBase (02/12/2024) e a data de admissão do colaborador:  
* DATEDIFF(DataBase, Dt_Admissao, DAY): Calcula a diferença de dias entre a data-base e a admissão.  
* DIVIDE(..., 7): Converte o número de dias em semanas.  
* INT(...): Retorna um número inteiro (descartando decimais).  
* +1: Considera a primeira semana como Semana 1.

Objetivo do Código:  
* Consolidar os dados de colaboradores efetivos e temporários admitidos a partir de dezembro de 2024.  
* Permitir a análise por semana de admissão para entender a evolução das contratações.

# Colunas Calculadas e Medidas
__Coluna Calculada - Tabela .Pessoas_TreinamentosPart__  
Código DAX:  
>.StatusIntegração =  
IF(  
    ISBLANK('.Pessoas_TreinamentosPart'[Assinatura]),  
    "Não Fez Integração Interna",  
    IF(  
        '.Pessoas_TreinamentosPart'[Assinatura] = "Sim",  
        "Assinado",  
        "Aguardando Assinatura"  
    )  
)

__Explicação do Código:__  
Categoriza o status da integração interna dos colaboradores com base na assinatura de participação.

A função IF é usada para definir três possíveis valores para .StatusIntegração:

* "Não Fez Integração Interna" → Se 'Assinatura' for BLANK(), significa que o colaborador não participou da integração.  
* "Assinado" → Se 'Assinatura' for "Sim", significa que o colaborador assinou a participação na integração.  
* "Aguardando Assinatura" → Se 'Assinatura' não for "Sim" nem BLANK(), significa que ele participou, mas ainda não assinou.

Objetivo do Código  
Essa coluna permite categorizar os colaboradores com relação à integração interna, possibilitando: ✅ Identificar quem ainda não fez a integração.  
* Verificar quem já assinou a presença.  
* Encontrar aqueles que fizeram, mas não assinaram.

__Medidas - Tabela .Pessoas_TreinamentosPart__  
Código DAX:  
>.TotalAguardando =  
CALCULATE(  
    DISTINCTCOUNT('.Pessoas_TreinamentosPart'[Colaborador]),  
    '.Pessoas_TreinamentosPart'[Assinatura] = "Não"  
)

>.TotalAprovados =  
CALCULATE(  
    DISTINCTCOUNT('.Pessoas_TreinamentosPart'[Colaborador]),  
    '.Pessoas_TreinamentosPart'[Assinatura] = "Sim"  
)

>.TotalSemIntegração =  
CALCULATE(  
    DISTINCTCOUNT('.Pessoas_TreinamentosPart'[Colaborador]),  
    ISBLANK('.Pessoas_TreinamentosPart'[Assinatura])  
)

__Explicação dos Códigos:__  
__Medida .TotalAguardando__  
Conta quantos colaboradores ainda não assinaram a participação na integração.

1. A medida usa CALCULATE e DISTINCTCOUNT para contar colaboradores únicos (DISTINCTCOUNT) que possuem 'Assinatura' = "Não".  
2. DISTINCTCOUNT('.Pessoas_TreinamentosPart'[Colaborador])  
    * Conta quantos colaboradores únicos existem na tabela .Pessoas_TreinamentosPart.  
3. CALCULATE(..., '.Pessoas_TreinamentosPart'[Assinatura] = "Não")  
    * Filtra apenas os colaboradores cujo campo 'Assinatura' é "Não".  
    * Assim, a contagem considera apenas os que ainda não assinaram.

Objetivo do Código  
* Retornar quantas pessoas ainda não assinaram a participação na integração.  
* Facilitar a gestão e acompanhamento de quem falta regularizar a assinatura.

__Medida .TotalAprovados__  
A medida usa CALCULATE e DISTINCTCOUNT para contar colaboradores únicos (DISTINCTCOUNT) que possuem 'Assinatura' = "Sim".

1. DISTINCTCOUNT('.Pessoas_TreinamentosPart'[Colaborador])  
    * Conta quantos colaboradores únicos existem na tabela .Pessoas_TreinamentosPart.  
2. CALCULATE(..., '.Pessoas_TreinamentosPart'[Assinatura] = "Sim")  
    * Filtra apenas os colaboradores cujo campo 'Assinatura' é "Sim", ou seja, aqueles que assinaram a participação.  
    * Assim, a contagem considera apenas os que já assinaram.

Objetivo do Código  
    * Retornar quantas pessoas já assinaram a participação na integração.  
    * Permitir monitoramento da taxa de aprovação/assinatura.

__Medida .TotalSemIntegração__  
A medida usa CALCULATE e DISTINCTCOUNT para contar colaboradores únicos (DISTINCTCOUNT) onde a coluna 'Assinatura' está vazia.

1. DISTINCTCOUNT('.Pessoas_TreinamentosPart'[Colaborador])  
    * Conta quantos colaboradores únicos existem na tabela .Pessoas_TreinamentosPart.  
2. CALCULATE(..., ISBLANK('.Pessoas_TreinamentosPart'[Assinatura]))  
    * Filtra apenas os colaboradores cujo campo 'Assinatura' está em branco (BLANK()).  
    * Assim, a contagem considera apenas os que não participaram da integração interna.

Objetivo do Código  
* Retornar quantas pessoas nunca participaram da integração interna.  
* Auxiliar no acompanhamento de colaboradores que ainda precisam ser treinados.

__Medidas - Tabela .Temporarios_Efetivos__  
Código DAX:  
>.AdmissõesPorSemanaEfetivo =  
CALCULATE(  
    COUNTROWS('.Temporarios_Efetivos'),  
    '.Temporarios_Efetivos'[Status] = "Efetivado"  
)

>.AdmissõesPorSemanaTemporario =  
CALCULATE(  
    COUNTROWS('.Temporarios_Efetivos'),  
    '.Temporarios_Efetivos'[Status] = "Temporário"  
)

>.ContratadosPorSemanaCalEfetivo =  
CALCULATE(  
    COUNT('.Temporarios_Efetivos'[Colaborador]),  
    ALLEXCEPT('.Temporarios_Efetivos', '.Temporarios_Efetivos'[Semana Admissão]),  
    '.Temporarios_Efetivos'[Status] = "Efetivado"  
)

>.ContratadosPorSemanaCalTemporário =  
CALCULATE(  
    COUNT('.Temporarios_Efetivos'[Colaborador]),  
    ALLEXCEPT('.Temporarios_Efetivos', '.Temporarios_Efetivos'[Semana Admissão]),  
    '.Temporarios_Efetivos'[Status] = "Temporário"  
)

>.DiferençaMeta1 =  
[.AdmissõesPorSemanaEfetivo] - [.MetaContratações1]

>.MetaContratações1 = 60

__Explicação do Código:__  
__Medida .AdmissõesPorSemanaEfetivo__  
Esse código DAX cria a medida .AdmissõesPorSemanaEfetivo, que calcula quantas pessoas foram admitidas como efetivas por semana.

1. A medida usa CALCULATE e COUNTROWS para contar quantas linhas existem na tabela .Temporarios_Efetivos onde o Status é "Efetivado".  
2. COUNTROWS('.Temporarios_Efetivos')  
    * Conta o número total de registros (linhas) na tabela .Temporarios_Efetivos.  
3. CALCULATE(..., '.Temporarios_Efetivos'[Status] = "Efetivado")  
    * Filtra apenas os registros onde Status seja "Efetivado".  
    * Assim, a contagem considera somente os colaboradores efetivados.

Objetivo do Código  
* Monitorar admissões de efetivos por semana.  
* Comparar efetivados com temporários ao longo do tempo.  
* Criar gráficos de tendência para entender padrões de contratação.

__Medida .AdmissõesPorSemanaTemporario__  
Esse código DAX cria a medida .AdmissõesPorSemanaTemporario, que calcula quantas pessoas foram admitidas como temporarias por semana.

1. A medida usa CALCULATE e COUNTROWS para contar quantas linhas existem na tabela .Temporarios_Efetivos onde o Status é "Temporario".  
2. COUNTROWS('.Temporarios_Efetivos')  
    * Conta o número total de registros (linhas) na tabela .Temporarios_Efetivos.  
3. CALCULATE(..., '.Temporarios_Efetivos'[Status] = "Temporario")  
    * Filtra apenas os registros onde Status seja "Temporario".  
    * Assim, a contagem considera somente os colaboradores temporarios.

Objetivo do Código  
* Monitorar admissões de temporario por semana.  
* Comparar efetivados com temporários ao longo do tempo.  
* Criar gráficos de tendência para entender padrões de contratação.

__Medida .ContratadosPorSemanaCalEfetivo__  
Esse código DAX cria a medida .ContratadosPorSemanaCalEfetivo, que calcula quantos colaboradores foram contratados como efetivos por semana.

1. COUNT('.Temporarios_Efetivos'[Colaborador])  
    * Conta o número total de colaboradores na tabela .Temporarios_Efetivos.  
2. ALLEXCEPT('.Temporarios_Efetivos', '.Temporarios_Efetivos'[Semana Admissão])  
    * Remove todos os filtros ativos, exceto o da coluna 'Semana Admissão'.  
    * Isso garante que os valores sejam calculados por semana.  
3. '.Temporarios_Efetivos'[Status] = "Efetivado"  
    * Filtra apenas os registros onde Status seja "Efetivado".  
    * Assim, a contagem considera somente os efetivados.

Objetivo do Código  
* Monitorar admissões semanais de efetivos.  
* Criar gráficos de tendência, analisando quantas pessoas foram efetivadas por semana.  
* Comparar admissões temporárias vs. efetivas ao longo do tempo.

__Medida .ContratadosPorSemanaCalTemporário__  
A medida usa CALCULATE para contar quantos colaboradores foram contratados como temporários, agrupando por Semana de Admissão.

1. COUNT('.Temporarios_Efetivos'[Colaborador])  
    * Conta o número total de colaboradores na tabela .Temporarios_Efetivos.  
2. ALLEXCEPT('.Temporarios_Efetivos', '.Temporarios_Efetivos'[Semana Admissão])  
    * Remove todos os filtros ativos, exceto o da coluna 'Semana Admissão'.  
    * Isso garante que os valores sejam calculados por semana.  
3. '.Temporarios_Efetivos'[Status] = "Temporário"  
    * Filtra apenas os registros onde Status seja "Temporário".  
    * Assim, a contagem considera somente os colaboradores temporários.

Objetivo do Código:  
* Monitorar admissões semanais de temporários.  
* Criar gráficos de tendência, analisando quantas pessoas foram contratadas como temporárias por semana.  
* Comparar admissões efetivas vs. temporárias ao longo do tempo.

__Medida .DiferençaMeta1__  
Esse código DAX cria a medida .DiferençaMeta1, que calcula a diferença entre o número de admissões efetivas semanais e a meta de contratações.  
A medida utiliza a subtração entre duas outras medidas: [.AdmissõesPorSemanaEfetivo] e [.MetaContratações1].

1. [.AdmissõesPorSemanaEfetivo]  
    * Esta medida conta quantos colaboradores foram efetivados em uma semana específica (como já discutido anteriormente).  
2. [.MetaContratações1]  
    * Esta medida representa a meta de contratações para aquela semana ou período. Pode ser um valor fixo ou uma meta calculada previamente.  
3. A operação "-" entre essas duas medidas calcula a diferença entre as admissões efetivas e a meta de contratações.

Objetivo do Código:  
1. Avaliar o desempenho de contratações em relação à meta estabelecida.  
2. A medida .DiferençaMeta1 pode indicar se a meta foi atingida (diferença positiva) ou não alcançada (diferença negativa).

__Medida .MetaContratações1__  
1. O código .MetaContratações1 = 60 define uma medida fixa que atribui o valor 60 como a meta de contratações.  
* MetaContratações1 é uma medida fixa que sempre terá o valor 60.  
* Não depende de filtros, contextos de dados ou cálculos dinâmicos.  
* Ela pode ser usada para comparação com outras medidas que calculam o número de contratações, como o número de admissões efetivas, ou em relatórios para mostrar a meta de contratações.

Objetivo do Código:  
* Estabelecer uma meta fixa de 60 contratações em um determinado período, como um mês ou ano.  
* Usada para comparar o número real de contratações com a meta estabelecida, ajudando a acompanhar o progresso da contratação.

__Medidas - Tabela EQUAT_INTEGRACAO__  
Código DAX:  
>.BernhoeftAguardando =  
COUNTROWS(  
    FILTER(  
        'EQUAT_INTEGRACAO',  
        'EQUAT_INTEGRACAO'[STATUS_BERNHOEFT] = "Análise"  
))

>.BernhoeftAprovado =  
COUNTROWS(  
    FILTER(  
        'EQUAT_INTEGRACAO',  
        'EQUAT_INTEGRACAO'[STATUS_BERNHOEFT] = "OK"  
))

>.ExternosAguardando =  
COUNTROWS(  
    FILTER(  
        'EQUAT_INTEGRACAO',  
        'EQUAT_INTEGRACAO'[INTEGRAÇÃO] = "AGUARDANDO INTEGRAÇÃO"  
    )  
)

>.ExternosAprovados =  
COUNTROWS(  
    FILTER(  
        'EQUAT_INTEGRACAO',  
        'EQUAT_INTEGRACAO'[INTEGRAÇÃO] = "APROVADO"  
    )  
)

>.RetornoBernhoeft =  
COUNTROWS(  
    FILTER(  
        'EQUAT_INTEGRACAO',  
        NOT(ISBLANK('EQUAT_INTEGRACAO'[STATUS_BERNHOEFT]))  
    )  
)

__Explicação do Código:__  
__Medida .BernhoeftAguardando__  
Conta o número de registros na tabela 'EQUAT_INTEGRACAO' onde o status do campo STATUS_BERNHOEFT é igual a "Análise".

1. FILTER('EQUAT_INTEGRACAO', 'EQUAT_INTEGRACAO'[STATUS_BERNHOEFT] = "Análise")  
    * Aplica um filtro na tabela 'EQUAT_INTEGRACAO', retornando apenas as linhas onde o campo 'STATUS_BERNHOEFT' tem o valor "Análise".  
2. COUNTROWS(...)  
    * Conta o número de linhas que atendem à condição do filtro.  
    * Ou seja, conta quantos registros estão aguardando análise com o status "Análise".

Objetivo do Código  
    * Monitorar o número de registros que estão em um status específico (no caso, "Análise").  
    * Pode ser usado para acompanhar o progresso de uma análise dentro de um processo, como em processos de validação, revisão ou aprovação.

__Medida .BernhoeftAprovado__  
Conta o número de registros na tabela 'EQUAT_INTEGRACAO' onde o status do campo STATUS_BERNHOEFT é igual a "OK".

1. A medida usa a função COUNTROWS em conjunto com FILTER para contar quantas linhas atendem à condição de ter o status "OK" no campo 'STATUS_BERNHOEFT'.  
2. FILTER('EQUAT_INTEGRACAO', 'EQUAT_INTEGRACAO'[STATUS_BERNHOEFT] = "OK")  
    * Aplica um filtro na tabela 'EQUAT_INTEGRACAO', retornando as linhas onde o campo 'STATUS_BERNHOEFT' tem o valor "OK".  
    * Isso significa que estamos interessados apenas nas linhas onde o status foi aprovado ou está em um estado final de "OK".  
3. COUNTROWS(...)  
    * Conta o número de linhas que passaram pelo filtro.  
    * Ou seja, conta quantos registros possuem o status "OK".

Objetivo do Código:  
    * Monitorar o número de registros aprovados ou que atingiram o status "OK" em um processo.  
    * Essa medida pode ser usada para acompanhar o número de processos que foram aprovados ou finalizados com sucesso.

__Medida .ExternosAguardando__  
Conta o número de registros na tabela 'EQUAT_INTEGRACAO' onde o campo INTEGRAÇÃO é igual a "AGUARDANDO INTEGRAÇÃO".

1. A medida usa a função COUNTROWS em conjunto com FILTER para contar quantas linhas da tabela 'EQUAT_INTEGRACAO' atendem a uma condição específica de status de integração.  
2. FILTER('EQUAT_INTEGRACAO', 'EQUAT_INTEGRACAO'[INTEGRAÇÃO] = "AGUARDANDO INTEGRAÇÃO")  
    * Aplica um filtro na tabela 'EQUAT_INTEGRACAO', retornando as linhas onde o campo INTEGRAÇÃO tem o valor "AGUARDANDO INTEGRAÇÃO".  
    * Isso significa que a medida está buscando os registros que ainda estão aguardando ser integrados.  
3. COUNTROWS(...)  
    * Conta o número de linhas que atendem à condição do filtro.  
    * Ou seja, a medida conta quantos registros estão com o status de integração aguardando.

Objetivo do Código:  
* Monitorar o número de registros aguardando integração.  
* Pode ser usado para acompanhar processos de integração que ainda não foram concluídos, ou seja, processos que estão pendentes.

__Medida .RetornoBernhoeft__  
Conta o número de registros na tabela 'EQUAT_INTEGRACAO' onde o campo STATUS_BERNHOEFT não está vazio (ou seja, não é nulo).

1. A medida usa as funções COUNTROWS, FILTER e ISBLANK para contar as linhas que atendem à condição de que o campo STATUS_BERNHOEFT não seja nulo ou em branco.  
2. FILTER('EQUAT_INTEGRACAO', NOT(ISBLANK('EQUAT_INTEGRACAO'[STATUS_BERNHOEFT])))  
    * Aplica um filtro na tabela 'EQUAT_INTEGRACAO' para retornar as linhas onde o campo STATUS_BERNHOEFT não está em branco.  
    * A função ISBLANK verifica se o campo 'STATUS_BERNHOEFT' é nulo ou em branco. O uso de NOT(ISBLANK(...)) garante que apenas as linhas com valores não nulos para esse campo sejam consideradas.  
3. COUNTROWS(...)  
    * Conta o número de linhas que atendem à condição de não terem o campo 'STATUS_BERNHOEFT' em branco.  
    * Ou seja, a medida retorna a quantidade de registros que têm algum valor no campo STATUS_BERNHOEFT.

Objetivo do Código:  
* Contabilizar os registros que possuem um valor válido no campo 'STATUS_BERNHOEFT'.  
* Pode ser útil para verificar quantos processos ou registros passaram por algum tipo de avaliação ou status (ou seja, foram preenchidos) no campo STATUS_BERNHOEFT.

__Colunas Calculadas - Tabela MOBILIZAÇÃO__  
Código DAX:  
>.CertificacaoStatus =  
IF(  
    OR(  
        ISBLANK('MOBILIZACAO'[CERT ELETRICISTA]),  
        ISBLANK('MOBILIZACAO'[NR 10 BÁSICO (INICIAL)])  
    ) ||  
    OR(  
        ISBLANK('MOBILIZACAO'[NR 10 BÁSICO (RECICLADO)]),  
        ISBLANK('MOBILIZACAO'[NR10 SEP (INICIAL)])  
    ) ||  
    OR(  
        ISBLANK('MOBILIZACAO'[NR10 SEP (RECICLADO)]),  
        ISBLANK('MOBILIZACAO'[NR 35 (INICIAL)])  
    ) ||  
    OR(  
        ISBLANK('MOBILIZACAO'[NR 35 (RECICLADO)]),  
        ISBLANK('MOBILIZACAO'[QUALIFICAÇÃO TÉCNICA])  
    ),  
    "Pendências",  
    IF(  
        AND(  
            TRIM(UPPER('MOBILIZACAO'[CERT ELETRICISTA])) = "OK",  
            TRIM(UPPER('MOBILIZACAO'[NR 10 BÁSICO (INICIAL)])) = "OK"  
        ) &&  
        AND(  
            TRIM(UPPER('MOBILIZACAO'[NR 10 BÁSICO (RECICLADO)])) = "OK",  
            TRIM(UPPER('MOBILIZACAO'[NR10 SEP (INICIAL)])) = "OK"  
        ) &&  
        AND(  
            TRIM(UPPER('MOBILIZACAO'[NR10 SEP (RECICLADO)])) = "OK",  
            TRIM(UPPER('MOBILIZACAO'[NR 35 (INICIAL)])) = "OK"  
        ) &&  
        AND(  
            TRIM(UPPER('MOBILIZACAO'[NR 35 (RECICLADO)])) = "OK",  
            TRIM(UPPER('MOBILIZACAO'[QUALIFICAÇÃO TÉCNICA])) = "OK"  
        ),  
        "Completo",  
        IF(  
            AND(  
                TRIM(UPPER('MOBILIZACAO'[CERT ELETRICISTA])) = "N/A",  
                TRIM(UPPER('MOBILIZACAO'[NR 10 BÁSICO (INICIAL)])) = "N/A"  
            ) &&  
            AND(  
                TRIM(UPPER('MOBILIZACAO'[NR 10 BÁSICO (RECICLADO)])) = "N/A",  
                TRIM(UPPER('MOBILIZACAO'[NR10 SEP (INICIAL)])) = "N/A"  
            ) &&  
            AND(  
                TRIM(UPPER('MOBILIZACAO'[NR10 SEP (RECICLADO)])) = "N/A",  
                TRIM(UPPER('MOBILIZACAO'[NR 35 (INICIAL)])) = "N/A"  
            ) &&  
            AND(  
                TRIM(UPPER('MOBILIZACAO'[NR 35 (RECICLADO)])) = "N/A",  
                TRIM(UPPER('MOBILIZACAO'[QUALIFICAÇÃO TÉCNICA])) = "N/A"  
            ),  
            "Completo",  
            IF(  
                OR(  
                    TRIM(UPPER('MOBILIZACAO'[CERT ELETRICISTA])) = "FALTA",  
                    TRIM(UPPER('MOBILIZACAO'[NR 10 BÁSICO (INICIAL)])) = "FALTA"  
                ) ||  
                OR(  
                    TRIM(UPPER('MOBILIZACAO'[NR 10 BÁSICO (RECICLADO)])) = "FALTA",  
                    TRIM(UPPER('MOBILIZACAO'[NR10 SEP (INICIAL)])) = "FALTA"  
                ) ||  
                OR(  
                    TRIM(UPPER('MOBILIZACAO'[NR10 SEP (RECICLADO)])) = "FALTA",  
                    TRIM(UPPER('MOBILIZACAO'[NR 35 (INICIAL)])) = "FALTA"  
                ) ||  
                OR(  
                    TRIM(UPPER('MOBILIZACAO'[NR 35 (RECICLADO)])) = "FALTA",  
                    TRIM(UPPER('MOBILIZACAO'[QUALIFICAÇÃO TÉCNICA])) = "FALTA"  
                ),  
                "Incompleto",  
                "Pendências"  
            )  
        )  
    )  
)

>.DocumentacaoStatus = 
IF(  
    OR(  
        ISBLANK('MOBILIZACAO'[CTPS]),  
        ISBLANK('MOBILIZACAO'[FICHA DE REGISTRO])  
    ) ||  
    OR(  
        ISBLANK('MOBILIZACAO'[RG/CPF]),  
        ISBLANK('MOBILIZACAO'[INTEGRAÇÃO])  
    ) ||  
    ISBLANK('MOBILIZACAO'[CNH]),  
    "Pendências",  
    IF(  
        AND(  
            TRIM(UPPER('MOBILIZACAO'[CTPS])) = "OK",  
            TRIM(UPPER('MOBILIZACAO'[FICHA DE REGISTRO])) = "OK"  
        ) &&  
        AND(  
            TRIM(UPPER('MOBILIZACAO'[RG/CPF])) = "OK",  
            TRIM(UPPER('MOBILIZACAO'[INTEGRAÇÃO])) = "OK"  
        ) &&  
        TRIM(UPPER('MOBILIZACAO'[CNH])) = "OK",  
        "Completo",  
        IF(  
            AND(  
                TRIM(UPPER('MOBILIZACAO'[CTPS])) = "N/A",  
                TRIM(UPPER('MOBILIZACAO'[FICHA DE REGISTRO])) = "N/A"  
            ) &&  
            AND(  
                TRIM(UPPER('MOBILIZACAO'[RG/CPF])) = "N/A",  
                TRIM(UPPER('MOBILIZACAO'[INTEGRAÇÃO])) = "N/A"  
            ) &&  
            TRIM(UPPER('MOBILIZACAO'[CNH])) = "N/A",  
            "Completo",  
            IF(  
                OR(  
                    TRIM(UPPER('MOBILIZACAO'[CTPS])) = "FALTA",  
                    TRIM(UPPER('MOBILIZACAO'[FICHA DE REGISTRO])) = "FALTA"  
                ),  
                "Incompleto",  
                IF(  
                    OR(  
                        TRIM(UPPER('MOBILIZACAO'[RG/CPF])) = "FALTA",  
                        TRIM(UPPER('MOBILIZACAO'[INTEGRAÇÃO])) = "FALTA"  
                    ),  
                    "Incompleto",  
                    IF(  
                        OR(  
                            TRIM(UPPER('MOBILIZACAO'[CNH])) = "FALTA",  
                            TRIM(UPPER('MOBILIZACAO'[CTPS])) = "N/A"  
                        ),  
                        "Incompleto",  
                        "Pendências"  
                    )  
                )  
            )  
        )  
    )  
)

__Explicação do Código:__  
__Coluna .CertificacaoStatus__  
Determina o status de certificação de um colaborador com base em várias colunas da tabela 'MOBILIZACAO'. Ele verifica se as certificações necessárias estão completas, faltando ou pendentes.

1. A medida avalia várias condições com base nos campos de certificação presentes na tabela 'MOBILIZACAO'. Ela verifica se os campos estão em branco, se possuem o status "OK", "N/A", "FALTA", ou outras condições, e retorna um status que pode ser "Pendências", "Completo", ou "Incompleto".  
2. Primeiro, verifica se há campos em branco:  
    * A função OR e ISBLANK verificam se qualquer um dos campos de certificação estão em branco. Se qualquer campo estiver em branco, o status será "Pendências".  
3. Se todos os campos estiverem preenchidos, verifica se todos estão com a certificação "OK":  
    * Se todos os campos de certificação tiverem o valor "OK", o status será "Completo".  
4. Se todos os campos estiverem com "N/A":  
    * Se todos os campos de certificação tiverem o valor "N/A", o status será também "Completo".  
5. Se houver campos com "FALTA":  
    * Se qualquer campo de certificação tiver o valor "FALTA", o status será "Incompleto".  
6. Se nenhum dos casos anteriores for atendido, o status será "Pendências".

Objetivo do Código:  
* Determinar o status de certificação de um colaborador com base em múltiplos campos de certificação.  
* A medida ajuda a identificar quais colaboradores estão com pendências de certificação, se as certificações estão completas, ou se há algum valor faltando.

__Coluna .DocumentacaoStatus__  
Avalia o status da documentação de um colaborador, levando em consideração várias informações sobre documentos específicos, como CTPS, Ficha de Registro, RG/CPF, Integração, e CNH. Dependendo dos valores desses campos, o código retorna um status indicando se a documentação está completa, incompleta ou se há pendências.

1. Primeiro, verifica se há campos em branco:  
    * A função OR e ISBLANK são usadas para verificar se qualquer um dos campos (CTPS, Ficha de Registro, RG/CPF, Integração, CNH) está em branco.  
    * Se qualquer campo estiver em branco, o status será "Pendências".  
2. Se todos os campos estão com "OK":  
    * Se todos os campos de documentação possuem o valor "OK", a documentação é considerada completa, e o status será "Completo".  
3. Se todos os campos estão com "N/A":  
    * Se todos os campos de documentação possuem o valor "N/A", isso também será considerado completo e o status será "Completo".  
4. Se algum campo está com "FALTA":  
    * Se qualquer campo tiver o valor "FALTA", o status será "Incompleto".  
    * O código verifica se qualquer combinação de campos de documentação está com o valor "FALTA". Se encontrar algum, o status será "Incompleto".  
5. Se nenhum dos casos anteriores for atendido:  
    * Se as condições de "OK", "N/A", ou "FALTA" não forem completamente atendidas, o status será "Pendências".

Objetivo do Código:  
* A medida .DocumentacaoStatus tem como objetivo avaliar e identificar o status da documentação de um colaborador, classificando-o em "Pendências", "Completo", ou "Incompleto" com base no preenchimento de determinados campos da tabela 'MOBILIZACAO'.

__Medidas__  
Código DAX:  
>.CertificadosOK =  
CALCULATE(  
    DISTINCTCOUNT(MOBILIZACAO[ID_Colaborador]),  
    MOBILIZACAO[NR 10 BÁSICO (INICIAL)] = "OK" &&  
    MOBILIZACAO[NR 10 BÁSICO (RECICLADO)] = "OK" &&  
    MOBILIZACAO[NR10 SEP (INICIAL)] = "OK" &&  
    MOBILIZACAO[NR10 SEP (RECICLADO)] = "OK" &&  
    MOBILIZACAO[NR 35 (INICIAL)] = "OK" &&  
    MOBILIZACAO[NR 35 (RECICLADO)] = "OK" &&  
    MOBILIZACAO[CERT ELETRICISTA] = "OK"  
)

>.DocumentacaoOK =  
CALCULATE(  
    DISTINCTCOUNT(MOBILIZACAO[ID_Colaborador]),  
    MOBILIZACAO[CTPS] = "OK" &&  
    MOBILIZACAO[FICHA DE REGISTRO] = "OK" &&  
    MOBILIZACAO[RG/CPF] = "OK" &&  
    MOBILIZACAO[INTEGRAÇÃO] = "OK" &&  
    MOBILIZACAO[CNH] = "OK"  
)

__Explicação do Código:__  
__Medida .CertificadosOK__  
Calcula a quantidade de colaboradores que possuem todos os certificados necessários com o status "OK". Essa medida é utilizada no Power BI para contar o número de colaboradores que têm os certificados de segurança e qualificação exigidos como "OK".

1. Função CALCULATE:  
A função CALCULATE é usada para modificar o contexto de filtro e realizar o cálculo desejado. No caso, ela está calculando a quantidade distinta de colaboradores que atendem a uma série de condições.

2. Função DISTINCTCOUNT:  
A função DISTINCTCOUNT conta a quantidade de valores distintos na coluna ID_Colaborador da tabela MOBILIZACAO. Isso significa que a medida irá contar quantos colaboradores (identificados pelo seu ID) têm todos os certificados com o status "OK".

3. Condições de Filtro:  
O filtro aplicado pela função CALCULATE está verificando se os seguintes campos têm o valor "OK":  
    * NR 10 BÁSICO (INICIAL)  
    * NR 10 BÁSICO (RECICLADO)  
    * NR10 SEP (INICIAL)  
    * NR10 SEP (RECICLADO)  
    * NR 35 (INICIAL)  
    * NR 35 (RECICLADO)  
    * CERT ELETRICISTA  
Se todos esses campos para um colaborador tiverem o valor "OK", esse colaborador será contado.

4. Resultado:  
A medida retorna o número total de colaboradores (contados de forma distinta) que têm todos os certificados exigidos com o status "OK". Ou seja, ela verifica se o colaborador possui os seguintes certificados, todos com o status "OK":  
    * NR 10 Básico (Inicial)  
    * NR 10 Básico (Reciclado)  
    * NR 10 SEP (Inicial)  
    * NR 10 SEP (Reciclado)  
    * NR 35 (Inicial)  
    * NR 35 (Reciclado)  
    * Certificação de Eletricista

Objetivo do Código:  
* A medida .CertificadosOK tem como objetivo contar a quantidade de colaboradores que estão com todos os certificados obrigatórios (como NR 10, NR 35, e Certificação de Eletricista) com o status "OK".

Isso é útil para garantir que todos os colaboradores estão devidamente qualificados conforme as exigências de segurança e certificação.

__Medida .DocumentacaoOK__  
Calcula o número de colaboradores que possuem toda a documentação necessária com o status "OK". A medida é utilizada no Power BI e conta o número de colaboradores que atenderam a todos os requisitos documentais especificados.

1. Função CALCULATE:  
A função CALCULATE é usada para modificar o contexto de filtro de uma expressão de medida e realizar o cálculo. Neste caso, ela está calculando o número de colaboradores que têm todos os documentos com o status "OK".

2. Função DISTINCTCOUNT:  
A função DISTINCTCOUNT conta a quantidade de valores distintos na coluna ID_Colaborador da tabela MOBILIZACAO. Ou seja, ela vai contar quantos colaboradores (identificados por seu ID) têm todos os documentos exigidos com o status "OK".

3. Condições de Filtro:  
O filtro aplicado pela função CALCULATE verifica se os seguintes campos da tabela MOBILIZACAO têm o valor "OK":  
    * CTPS: Documento de Carteira de Trabalho e Previdência Social  
    * FICHA DE REGISTRO: Documento de ficha de registro de empregado  
    * RG/CPF: Documento de identidade ou Cadastro de Pessoas Físicas  
    * INTEGRAÇÃO: Campo que possivelmente se refere à documentação relacionada à integração do colaborador  
    * CNH: Carteira Nacional de Habilitação  
Se todos esses campos tiverem o valor "OK", o colaborador será contado.

4. Resultado:
A medida retorna o número total de colaboradores que têm todos os documentos exigidos (CTPS, Ficha de Registro, RG/CPF, Integração e CNH) com o status "OK".

Objetivo do Código:  
* A medida .DocumentacaoOK tem o objetivo de garantir que somente os colaboradores com toda a documentação exigida em conformidade (com status "OK") sejam contados. Isso é útil para verificar se todos os colaboradores estão legalmente e administrativamente adequados para realizar suas funções na empresa, com todos os documentos essenciais para a contratação e o exercício do trabalho.

__Medidas - Tabela PESSOAS__  
Código DAX: 
>.AdmissõesPorSemana =  
COUNTROWS(  
    'PESSOAS'  
)

>.ContratadosPorSemanaCal =  
CALCULATE(  
    COUNT(PESSOAS[Colaborador]),  
    ALLEXCEPT(PESSOAS, PESSOAS[Semana Admissão])  
)

>.DiferençaMeta =  
[.AdmissõesPorSemana] - [.MetaContratações]

>.MetaContratações = 60

__Explicação do Código:__  
__Medida .AdmissõesPorSemana__  
Conta o número total de admissões registradas na tabela PESSOAS. Ela usa a função COUNTROWS para contar as linhas da tabela PESSOAS, sem nenhum filtro adicional, ou seja, ela conta o número total de registros de colaboradores na tabela, sem discriminação por data ou outros critérios.

1. Função COUNTROWS:  
A função COUNTROWS conta o número de linhas em uma tabela ou em uma expressão que retorna uma tabela. Neste caso, ela conta o número de registros (linhas) na tabela PESSOAS.

2. Tabela PESSOAS:  
A tabela PESSOAS provavelmente contém informações sobre os colaboradores, incluindo dados como ID, Nome, Data de Admissão, entre outros. Não há filtros aplicados, então todas as linhas da tabela serão contadas.

3. Resultado:  
A medida retorna o número total de colaboradores ou admissões presentes na tabela PESSOAS. Ou seja, ela não filtra ou segmenta as admissões por semana ou outro critério; simplesmente conta todas as linhas da tabela.

Objetivo do Código:  
* A medida .AdmissõesPorSemana serve para contar o total de admissões ou registros de colaboradores na tabela PESSOAS. Caso queira contar admissões por semana, mês ou outro período específico, seria necessário aplicar um filtro ou utilizar uma função de agrupamento com base na data de admissão.

__Medida .ContratadosPorSemanaCal__  
Calcula o número de colaboradores contratados por semana, considerando a tabela PESSOAS. Vamos entender como essa medida funciona detalhadamente.

1. Função CALCULATE:  
* A função CALCULATE é uma das mais importantes no DAX e permite alterar o contexto de cálculo de uma medida. Ela avalia uma expressão, podendo modificar o contexto de linha ou de filtro. No caso dessa medida, a expressão que ela avalia é COUNT(PESSOAS[Colaborador]), ou seja, ela conta o número de colaboradores na tabela PESSOAS com base no contexto modificado.

2. Função COUNT:  
* A função COUNT conta o número de valores não nulos na coluna especificada. Neste caso, PESSOAS[Colaborador] é a coluna usada, então a medida vai contar o número de colaboradores (ou o número de valores não nulos na coluna Colaborador).

3. Função ALLEXCEPT:  
* A função ALLEXCEPT é usada para remover todos os filtros da tabela, exceto os filtros que se referem à(s) coluna(s) especificada(s). Nesse caso, ALLEXCEPT(PESSOAS, PESSOAS[Semana Admissão]) mantém o filtro da coluna Semana Admissão e remove os filtros de todas as outras colunas da tabela PESSOAS.  
* Isso significa que a contagem de colaboradores será feita de forma agrupada por semana de admissão, mantendo o contexto da coluna Semana Admissão intacto. Ou seja, os colaboradores serão contados para cada semana de admissão, mas os demais filtros aplicados no modelo (como filtros por data, status, etc.) serão ignorados.

4. Resultado da Medida:  
* O objetivo dessa medida é calcular o número de colaboradores contratados por semana. Ela conta o número de colaboradores na tabela PESSOAS, agrupando por Semana Admissão, e ignora qualquer outro filtro que possa estar presente nas outras colunas.

Objetivo do Código:  
* A medida .ContratadosPorSemanaCal é útil para calcular e visualizar o número de colaboradores contratados por semana. Ela agrupa os dados por semana de admissão e ignora outros filtros no modelo, proporcionando uma visão clara de quantos colaboradores foram contratados em cada semana, sem distorções por outros critérios de filtro.

__Medida .DiferençaMeta__  
Calcula a diferença entre o número total de admissões por semana e a meta de contratações. Essa medida é útil para comparar o desempenho real de admissões com a meta estabelecida, permitindo identificar se a meta está sendo atingida ou não.

Componentes:  
1. [.AdmissõesPorSemana]:  
    * Essa medida (presumivelmente criada anteriormente) calcula o número total de admissões por semana. Em outras palavras, ela conta quantos colaboradores foram admitidos durante uma semana específica.  
2. [.MetaContratações]:  
    * Essa medida (ou variável) representa a meta de contratações definida para um determinado período, geralmente um valor numérico fixo ou uma meta estabelecida pela empresa para o número de admissões.  
    * No exemplo anterior, a meta de contratações pode ser um número fixo como 60, ou pode ser calculada de forma dinâmica dependendo de outros fatores.  
3. Operação de Subtração:  
    * A medida .DiferençaMeta calcula a diferença entre o número real de admissões (.AdmissõesPorSemana) e a meta de contratações (.MetaContratações).  
    * O resultado será um número que representa quanto a quantidade real de admissões está acima ou abaixo da meta:  
    * Se o número de admissões for maior que a meta, o resultado será positivo, indicando que a meta foi superada.  
    * Se o número de admissões for menor que a meta, o resultado será negativo, indicando que a meta não foi atingida.  

Objetivo da Medida:  
* A medida .DiferençaMeta é usada para comparar o desempenho real de admissões com a meta, proporcionando uma visão clara do desvio em relação à meta. Isso ajuda a tomar decisões e ajustar estratégias, como contratar mais pessoas ou melhorar processos, dependendo de como a meta está sendo atendida.

__Medida .MetaContratações__  
Define um valor fixo para a meta de contratações. Neste caso, a meta de contratações é 60, o que significa que a empresa ou equipe de recursos humanos definiu como objetivo contratar 60 colaboradores em um período específico, como uma semana, mês, ou outro intervalo de tempo relevante.  
    * .MetaContratações é o nome da medida ou variável.  
    * O valor 60 é atribuído diretamente à medida, indicando que a meta de contratações para o período é de 60 pessoas.

1. Objetivo de Contratação:  
    * A medida .MetaContratações serve como referência ou objetivo para o número de pessoas que devem ser contratadas.  
    * Esse valor pode ser usado em várias análises, como comparar o número real de admissões com a meta (como na medida .DiferençaMeta, onde comparamos o valor real de admissões com a meta definida).  
2. Uso em Cálculos:  
    * A medida .MetaContratações pode ser usada em outras fórmulas de cálculo, como:  
        * Comparar se o número de contratações atingiu, superou ou ficou abaixo da meta.  
        * Ajustar planos de recrutamento ou recursos com base na diferença entre contratações reais e a meta.

# Indicadores KPIs
O painel é interativo e conta com segmentadores de dados por unidade, função e data de admissão.

__Gráficos Utilizados__  
1. Cartão de Contagem Distinta de Colaboradores  
Exibe a quantidade total de colaboradores considerados no relatório.

2. Gráfico de Rosca - Distribuição de Status de Certificação  
Legenda: Coluna .CertificacaoStatusValores: Contagem da coluna .CertificacaoStatus  
Valores: Contagem da coluna .CertificacaoStatus  
Lógica de Cálculo:  
    * "Pendências": Se qualquer dos campos de certificação estiver em branco.  
    * "Completo": Se todos os campos contiverem "OK" ou "N/A".  
    * "Incompleto": Se qualquer campo contiver "FALTA".

3. Gráfico de Rosca - Distribuição de Status de Documentação  
Legenda: Coluna .DocumentacaoStatusValores: Contagem da coluna .DocumentacaoStatus  
Valores: Contagem da coluna .DocumentacaoStatus  
Lógica de Cálculo:  
    * "Pendências": Se qualquer documento obrigatório estiver em branco.  
    * "Completo": Se todos os documentos estiverem como "OK" ou "N/A".  
    * "Incompleto": Se qualquer documento estiver como "FALTA".

4. Gráfico de Barras Clusterizado - Porcentagem de Colaboradores por Função  
Eixo Y: Coluna FunçãoEixo X: Contagem da coluna Colaborador  
Eixo X: Contagem da coluna Colaborador

5. Gráfico de Colunas Clusterizado - Distribuição de Colaboradores por Base e ASO  
Eixo Y: Coluna Base (Cidades)Eixo X: Contagem distinta da coluna ColaboradorLegenda: Coluna ASO  
Eixo X: Contagem distinta da coluna Colaborador

6. Gráfico de Matriz - Progresso de Certificações e Entrega de EPIs
Linhas: Coluna ColaboradorColunas: Em brancoValores: Colunas Certificados, Documentos e EPI
Configuração especial de elementos da célula:
    * "Falta" → ícone X
    * "Pendências" → ícone !
    * "OK" → ícone V
    * "Assinar" → ícone !

Legenda dos Ícones
X = Falta Itens
! = Pendências
V = OK