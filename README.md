
# Sistema AutoGestão - Gerenciamento de Banco de Dados

Este repositório contém a documentação técnica e os tópicos da apresentação do projeto **AutoGestão**, um sistema robusto desenvolvido para a modernização e automação de operações em locadoras de veículos.

## 📋 Tópicos da Apresentação (Slides de Defesa)

A apresentação foi estruturada para fornecer uma visão executiva e técnica do projeto:

1.  **Apresentação Geral:** Definição do que é o AutoGestão e seu propósito de automatizar processos manuais.
2.  **Caracterização do Problema:** Identificação das falhas em sistemas manuais ou obsoletos na gestão de frotas.
3.  **Justificativa Técnica:** Por que a implementação de um banco de dados relacional é a solução ideal.
4.  **Objetivos:**
    * **Geral:** Modernizar as operações da locadora.
    * **Específicos:** Controle de frotas, automação de cálculos de diárias e multas, e garantia de integridade de dados.
5.  **Metas e Fases de Execução:** Cronograma de desenvolvimento (Abril/2026 a Maio/2026).
6.  **Metodologia e Orçamento:** Ferramentas open-source utilizadas e abordagem de desenvolvimento.
7.  **Resultados e Considerações Finais:** Benefícios alcançados, como eliminação de falhas humanas e otimização de receita.
8.  **Referências:** Bases teóricas e ferramentas (MySQL, phpMyAdmin, Trello, GitHub).



## 📄 Tópicos do Relatório Técnico (Documentação)

O relatório detalha a implementação técnica e a engenharia por trás do sistema:

### 1. Introdução e Planejamento
* Contextualização do projeto no curso Técnico em Informática.
* Descrição do ambiente de aprendizagem (Laboratório de Informática).

### 2. Levantamento e Análise de Requisitos
* Definição de Requisitos Funcionais (ex: cadastro de veículos, registro de locações).
* Definição de Requisitos Não Funcionais (ex: segurança de dados, performance do SGBD).

### 3. Modelagem de Dados
* **Modelo Conceitual:** Diagramas de Entidade-Relacionamento (DER).
* **Modelo Lógico:** Estrutura de tabelas e relacionamentos (Chaves Primárias e Estrangeiras).

### 4. Implementação SQL
* **DML (Data Manipulation Language):** Scripts de inserção e carga inicial de dados.
* **Automações e Regras de Negócio:**
    * **Stored Procedures:** Lógica para cálculo automático de valores e processamento de devoluções.
    * **Triggers:** Verificações automáticas de integridade.
    * **Views:** Relatórios consolidados como "Clientes Mais Ativos" e "Receita por Veículo".

### 5. Gestão e Ferramentas
* Uso do **Trello** para metodologias ágeis.
* **Git/GitHub** para controle de versão.
* **phpMyAdmin** para administração do banco.

---
# 🗄️ Documentação Técnica: Banco de Dados AutoGestão

Este documento detalha a arquitetura do banco de dados relacional **AutoGestão**, incluindo a estrutura das tabelas, estratégias de povoamento, automações via triggers e relatórios gerenciais através de views.

---

## 🏗️ 1. Estrutura das Tabelas (DDL)
Abaixo estão os scripts de criação das **9 tabelas** do sistema. Esta estrutura foi planejada para garantir a integridade referencial e o suporte completo às regras de negócio da locadora.

* **Tabela `Administradores`**: Gere o acesso ao sistema, distinguindo permissões entre os níveis 'Super' e 'Gerente'.
* **Tabela `Categorias`**: Define os perfis dos veículos (Económico, SUV, Luxo) e estabelece o valor base da diária.
* **Tabela `Seguros`**: Armazena as modalidades de proteção e coberturas disponíveis para contratação.
* **Tabela `Clientes`**: Registo detalhado dos locatários, com controlo rigoroso de CPF e validade da CNH.
* **Tabela `Funcionários`**: Registo dos colaboradores que operam o sistema e realizam os atendimentos.
* **Tabela `Veículos`**: Controlo da frota ativa, vinculando cada unidade à sua categoria e monitorizando a disponibilidade.
* **Tabela `Locações`**: Registo central dos contratos, unindo cliente, veículo, seguro e funcionário.
* **Tabela `Devoluções`**: Processa o encerramento das locações e calcula o acerto financeiro final.
* **Tabela `Manutenções`**: Histórico preventivo e corretivo, essencial para a segurança e conservação da frota.

---
1. Tabela de `Administradores` 

```sql
CREATE TABLE Administradores 
( id_admin INT AUTO_INCREMENT PRIMARY KEY, 
nome VARCHAR(100) NOT NULL, email VARCHAR(100) NOT NULL UNIQUE, 
senha_hash VARCHAR(255) NOT NULL, 
nivel_acesso ENUM('Super', 'Gerente') DEFAULT 'Gerente',
data_criacao DATETIME DEFAULT CURRENT_TIMESTAMP ) ENGINE=InnoDB;


2. Tabela de `Categorias` 

CREATE TABLE Categorias ( id_categoria INT AUTO_INCREMENT PRIMARY KEY, 
nome_categoria VARCHAR(50) NOT NULL UNIQUE,
valor_diaria_base DECIMAL(10, 2) NOT NULL ) ENGINE=InnoDB;

3. Tabela de `Seguros`  

CREATE TABLE `Seguros` ( id_seguro INT AUTO_INCREMENT PRIMARY KEY, 
tipo_cobertura VARCHAR(50) NOT NULL UNIQUE,
valor_diario_seguro DECIMAL(10, 2) NOT NULL ) ENGINE=InnoDB;

4. Tabela de `Clientes` 

CREATE TABLE Clientes ( id_cliente INT
AUTO_INCREMENT PRIMARY KEY, nome VARCHAR(100) NOT NULL, cpf VARCHAR(14)
NOT NULL UNIQUE, cnh_numero VARCHAR(20) NOT NULL UNIQUE, cnh_vencimento
DATE NOT NULL, telefone VARCHAR(20) ) ENGINE=InnoDB;

5. Tabela de `Funcionários` 

CREATE TABLE Funcionarios (
id_funcionario INT AUTO_INCREMENT PRIMARY KEY, nome VARCHAR(100) NOT NULL,
matricula VARCHAR(20) NOT NULL UNIQUE, cargo VARCHAR(50) NOT NULL )
ENGINE=InnoDB;

6. Tabela de `Veículos`  

CREATE TABLE Veiculos ( id_veiculo
INT AUTO_INCREMENT PRIMARY KEY, placa VARCHAR(10) NOT NULL UNIQUE, modelo
VARCHAR(50) NOT NULL, cor VARCHAR(30) NOT NULL, ano INT NOT NULL, status
ENUM('Disponível', 'Locado', 'Manutenção') DEFAULT 'Disponível', fk_id_categoria INT
NOT NULL, CONSTRAINT fk_veiculo_categoria FOREIGN KEY (fk_id_categoria)
REFERENCES Categorias(id_categoria) ) ENGINE=InnoDB;

7. Tabela de `Locações` 
CREATE TABLE Locacoes ( id_locacao INT AUTO_INCREMENT PRIMARY KEY,
data_retirada DATETIME NOT NULL, data_prevista_devolucao DATETIME NOT NULL,
fk_id_cliente INT NOT NULL, fk_id_veiculo INT NOT NULL, fk_id_funcionario INT NOT
NULL, fk_id_seguro INT NOT NULL, CONSTRAINT fk_locacao_cliente FOREIGN KEY
(fk_id_cliente) REFERENCES Clientes(id_cliente), CONSTRAINT fk_locacao_veiculo
FOREIGN KEY (fk_id_veiculo) REFERENCES Veiculos(id_veiculo), CONSTRAINT
fk_locacao_funcionario FOREIGN KEY (fk_id_funcionario) REFERENCES
Funcionarios(id_funcionario), CONSTRAINT fk_locacao_seguro FOREIGN KEY
(fk_id_seguro) REFERENCES Seguros(id_seguro) ) ENGINE=InnoDB;


8. Tabela de `Devoluções` 

CREATE
TABLE Devolucoes ( id_devolucao INT AUTO_INCREMENT PRIMARY KEY,
data_entrega_real DATETIME NOT NULL, valor_total_pago DECIMAL(10, 2) NOT NULL,
valor_multa DECIMAL(10, 2) DEFAULT 0.00, fk_id_locacao INT NOT NULL UNIQUE,
CONSTRAINT fk_devolucao_locacao FOREIGN KEY (fk_id_locacao) REFERENCES
Locacoes(id_locacao) ) ENGINE=InnoDB;


9. Tabela de `Manutenções` 

CREATE TABLE Manutencoes 
(id_manutencao INT AUTO_INCREMENT PRIMARY KEY, descricao TEXT NOT NULL,
data_inicio DATE NOT NULL, data_fim DATE, custo_reparo DECIMAL(10, 2),
fk_id_veiculo INT NOT NULL, CONSTRAINT fk_manutencao_veiculo FOREIGN KEY
(fk_id_veiculo) REFERENCES Veiculos(id_veiculo) ) ENGINE=InnoDB;**
```
---

## 📥 2. Povoamento do Banco (DML - Inserts)
Esta seção contém os dados iniciais necessários para colocar o sistema em operação e validar os relacionamentos entre as tabelas.

* **Descrição Geral:** Os scripts abaixo realizam a carga inicial de categorias de mercado, veículos de exemplo, tipos de seguros padrão e registos fictícios de clientes e funcionários para testes de fluxo e validação do sistema.
---
```sql
1. Inserindo Administradores 
    INSERT INTO Administradores (nome, email, senha_hash, nivel_acesso) 
    VALUES ('Carlos Mendes', 'carlos@autogestao.com', 'hash_123', 'Super');

2. Inserindo Categorias 
    INSERT INTO Categorias (nome_categoria, valor_diaria_base) 
    VALUES ('Hatch Econômico', 80.00);

3. Inserindo Seguros 
    INSERT INTO Seguros (tipo_cobertura, valor_diario_seguro) VALUES ('Básico', 20.00);

4. Inserindo Clientes (CNH validade futura) 
    INSERT INTO Clientes (nome, cpf, cnh_numero, cnh_vencimento, telefone) 
    VALUES ('João Pereira', '111.111.111-01', 'CNH00000001', '2028-05-10', '99999-0001');

5. Inserindo Funcionários 
    INSERT INTO Funcionarios (nome, matricula, cargo)
    VALUES ('Wesley Neves Batista', 'MAT001', 'Desenvolvedor/Suporte')

6. Inserindo 20 Veículos (Ligando à fk_id_categoria de 1 a 20) 
    INSERT INTO Veiculos (placa, modelo, cor, ano, status, fk_id_categoria) 
    VALUES ('ABC-1234', 'Chevrolet Onix',
'Branco', 2024, 'Disponível', 1);

7. Inserindo Locações (Abril de 2026) 
    INSERT INTO Locacoes (data_retirada, data_prevista_devolucao, fk_id_cliente, fk_id_veiculo, fk_id_funcionario, fk_id_seguro)
    VALUES ('2026-04-01 08:00:00', '2026-04-05 08:00:00', 1, 1, 2, 1);

8. Inserindo Devoluções 
    INSERT INTO Devolucoes (data_entrega_real, valor_total_pago, valor_multa, fk_id_locacao) 
    VALUES ('2026-04-05 08:00:00', 400.00, 0.00, 1);

9. Inserindo Manutenções 
    INSERT INTO Manutencoes (descricao, data_inicio, data_fim, custo_reparo, fk_id_veiculo) 
    VALUES ('Troca de óleo e filtro', '2026-01-10', '2026-01-10', 250.00, 1);

```

## ⚙️ 3. Regras de Integridade e Gatilhos (Triggers)

Os gatilhos automáticos garantem que o banco de dados mantenha a consistência operacional sem necessidade de intervenção manual, reagindo instantaneamente a cada movimentação na frota.

* **Indisponibilidade por Locação**: Altera o estado do veículo para 'Locado' assim que um contrato é aberto, evitando reservas duplicadas.
* **Retorno à Disponibilidade**: Libera o veículo automaticamente para novos clientes assim que o registro de devolução é finalizado.
* **Controle de Manutenção**: Garante que veículos em reparo fiquem marcados como 'Manutenção', impedindo sua locação indevida.
 ```sql
1. TRIGGER: Bloquear o veículo ao registrar uma locação

DELIMITER // CREATE TRIGGER trg_atualiza_status_locacao AFTER INSERT ON Locacoes FOR EACH ROW BEGIN 
UPDATE Veiculos SET status = 'Locado' WHERE id_veiculo = NEW.fk_id_veiculo; END //
DELIMITER ;

    INSERT INTO Locacoes (data_retirada, data_prevista_devolucao, fk_id_cliente,
fk_id_veiculo, fk_id_funcionario, fk_id_seguro) 
    VALUES ('2026-05-01 10:00:00', '2026-
05-05 10:00:00', 1, 1, 1, 1);

    INSERT INTO Devolucoes (data_entrega_real, valor_total_pago, valor_multa, fk_id_locacao) 
    VALUES ('2026-05-05 10:00:00', 500.00, 0.00, 1);

    SELECT id_veiculo, placa, status FROM Veiculos WHERE id_veiculo = 1;


2. TRIGGER: Liberar o veículo ao registrar a devolução

DELIMITER // CREATE TRIGGER trg_atualiza_status_devolucao AFTER INSERT ON Devolucoes FOR EACH ROW BEGIN
o status de volta para 'Disponível' 
UPDATE Veiculos SET status = 'Disponível' WHERE
id_veiculo = (SELECT fk_id_veiculo FROM Locacoes WHERE id_locacao = NEW.fk_id_locacao); END // DELIMITER ;

3. TRIGGER: Bloquear o veículo ao enviar para manutenção

DELIMITER // CREATE TRIGGER trg_atualiza_status_manutencao AFTER INSERT ON
Manutencoes FOR EACH ROW BEGIN  
UPDATE Veiculos SET status = 'Manutenção' WHERE id_veiculo =
NEW.fk_id_veiculo; END // DELIMITER ;

```
---

## ⚙️ 4. Procedimentos Armazenados (Stored Procedures)

As Procedures encapsulam a lógica financeira e operacional, permitindo que processos complexos de cálculo sejam executados com segurança e padronização.

* **`sp_registrar_devolucao`**: Automatiza o encerramento do contrato. Ela realiza o cruzamento de datas, calcula o valor das diárias, aplica multas proporcionais em caso de atraso e consolida o saldo financeiro final na tabela de Devoluções.

```sql
DELIMITER // CREATE PROCEDURE sp_registrar_devolucao( IN p_id_locacao INT, IN
p_data_entrega_real DATETIME ) BEGIN -- Declaração das variáveis para armazenar os
dados da locação DECLARE v_data_prevista DATETIME; DECLARE v_data_retirada
    DATETIME; DECLARE v_valor_diaria_carro DECIMAL(10,2); DECLARE
v_valor_diario_seguro DECIMAL(10,2);
    DECLARE v_dias_previstos INT;
    DECLARE v_dias_reais INT;
    DECLARE v_dias_atraso INT DEFAULT 0;
    DECLARE v_valor_total_previsto DECIMAL(10,2);
    DECLARE v_valor_multa DECIMAL(10,2) DEFAULT 0.00;
    DECLARE v_valor_final_pago DECIMAL(10,2);

1. Busca os dados do contrato, do veículo (categoria) e do seguro

    SELECT
 l.data_retirada,
 l.data_prevista_devolucao,
 c.valor_diaria_base,
 s.valor_diario_seguro
    INTO
 v_data_retirada, v_data_prevista, v_valor_diaria_carro,
v_valor_diario_seguro
    FROM Locacoes l
    JOIN Veiculos v ON l.fk_id_veiculo = v.id_veiculo
    JOIN Categorias c ON v.fk_id_categoria = c.id_categoria
    JOIN Seguros s ON l.fk_id_seguro = s.id_seguro
    WHERE l.id_locacao = p_id_locacao;

2. Calcula a quantidade de dias previstos no contrato.

    SET v_dias_previstos = DATEDIFF(v_data_prevista, v_data_retirada);
    IF v_dias_previstos = 0 THEN SET v_dias_previstos = 1; END IF; 

3. Calcula o valor total previsto original (sem multas)

    SET v_valor_total_previsto = (v_valor_diaria_carro +
v_valor_diario_seguro) * v_dias_previstos;

4. Verifica se houve atraso
    
    IF p_data_entrega_real > v_data_prevista THEN
    SET v_dias_atraso = DATEDIFF(p_data_entrega_real,
v_data_prevista);

    SET v_valor_multa = (v_valor_total_previsto * 0.10) +
((v_valor_diaria_carro + v_valor_diario_seguro) * v_dias_atraso);
    END IF;

5. Calcula o valor final que o cliente deve pagar

    SET v_valor_final_pago = v_valor_total_previsto + v_valor_multa;

6. Insere o registro definitivo na tabela de Devoluções
    
    INSERT INTO Devolucoes (data_entrega_real, valor_total_pago,
valor_multa, fk_id_locacao)
    VALUES (p_data_entrega_real, v_valor_final_pago, v_valor_multa,
p_id_locacao);
    END //
    DELIMITER ;
    call sp_registrar_devolucao(2, '2026-05-10 14:00:00');


```

---

## 📊 5. Inteligência de Dados e Relatórios (Views)
As visualizações facilitam a extração de métricas estratégicas para a tomada de decisão gerencial:

* **`vw_faturamento_mensal`**: Consolida o montante total arrecadado e o volume de contratos encerrados, agrupados por mês.
* **`vw_controle_multas`**: Relatório focado na arrecadação mensal proveniente de penalidades por atraso.
* **`vw_rentabilidade_veiculo`**: Ranking de performance dos veículos, destacando os modelos mais lucrativos através do cruzamento de locações e receita gerada.
* **`vw_clientes_mais_ativos`**: Identificação dos clientes fidelizados por volume de utilização e total investido.

```sql

1. VIEW : Faturamento Mensal 

    CREATE VIEW vw_faturamento_mensal AS SELECT DATE_FORMAT(data_entrega_real,
'%Y-%m') AS mes_referencia, COUNT(id_devolucao) 
    AS total_locacoes_encerradas,
SUM(valor_total_pago) AS faturamento_total FROM Devolucoes    
    GROUP BY
    DATE_FORMAT(data_entrega_real, '%Y-%m') 
    ORDER BY mes_referencia DESC;

2. VIEW: Controle de Multas Aplicadas

    CREATE VIEW vw_controle_multas AS SELECT DATE_FORMAT(data_entrega_real, '%Y-%m') 
    AS mes_referencia, COUNT(id_devolucao) 
    AS quantidade_multas_aplicadas, SUM(valor_multa) 
    AS total_multas_arrecadadas 
    FROM Devolucoes WHERE valor_multa > 0 
    GROUP BY DATE_FORMAT(data_entrega_real, '%Y-%m') 
    ORDER BY mes_referencia DESC;

3. VIEW: Rentabilidade por Veículo (Mais e Menos Alugados) 

    CREATE VIEW vw_rentabilidade_veiculo AS SELECT v.placa, v.modelo, c.nome_categoria, COUNT(l.id_locacao) 
    AS total_vezes_alugado, IFNULL(SUM(d.valor_total_pago), 0) 
    AS receita_total_gerada 
    FROM Veiculos v JOIN Categorias c ON v.fk_id_categoria = c.id_categoria 
    LEFT JOIN Locacoes l ON
v.id_veiculo = l.fk_id_veiculo 
    LEFT JOIN Devolucoes d ON l.id_locacao = d.fk_id_locacao 
    GROUP BY v.id_veiculo, v.placa, v.modelo, c.nome_categoria 
    ORDER BY receita_total_gerada DESC;

4. VIEW: Clientes Mais Ativos

    CREATE VIEW vw_clientes_mais_ativos AS SELECT cl.nome 
    AS cliente_nome, cl.cpf,
COUNT(l.id_locacao) AS quantidade_locacoes, IFNULL(SUM(d.valor_total_pago), 0) 
    AS valor_total_gasto 
    FROM Clientes cl 
    LEFT JOIN Locacoes l ON cl.id_cliente =
l.fk_id_cliente 
    LEFT JOIN Devolucoes d ON l.id_locacao = d.fk_id_locacao GROUP BY cl.id_cliente, cl.nome, cl.cpf 
    ORDER BY quantidade_locacoes DESC, valor_total_gasto
DESC;

```
---
*Projeto desenvolvido como parte da Prática Profissional Supervisionada - CETAM.*
---
*Projeto desenvolvido por: Ana Carolina Bomfim, Breno Giovanni Frota, Marley Gabriel Alves e Wesley Neves.*
