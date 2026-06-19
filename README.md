# LiftLogic 🛗

O **LiftLogic** é um sistema inteligente de coordenação e simulação de elevadores em tempo real para um prédio de 11 andares (0 ao 10). Desenvolvido em Python via CLI (Interface de Linha de Comando), o projeto aplica conceitos de Programação Orientada a Objetos (POO) para gerenciar duas cabines de forma otimizada e segura.

---

## 📌 1. Introdução

### 1.1 Propósito do Projeto
Este documento e código especificam os requisitos funcionais e não funcionais para o Sistema de um Elevador seguindo as diretrizes de Engenharia de Software inspiradas na norma **IEEE 29148:2018**.

### 1.2 Escopo
O SE (Sistema de Elevador) permite que os usuários se desloquem pelo prédio de forma organizada, diminuindo o tempo de espera através de lógica de proximidade e garantindo a segurança por meio de travas de superlotação.

### 1.3 Acrônimos
* **SE:** Sistema de Elevador
* **RF:** Requisito Funcional
* **RNF:** Requisito Não Funcional
* **RN:** Regra de Negócio
* **Sprint:** Unidade de iteração/aula do projeto

---

## 📋 2. Requisitos e Regras de Negócio

### 2.1 Requisitos Funcionais (RF)

* **RF-001: Andar Desejado**
    * *Descrição:* O sistema deve fornecer uma interface para que o usuário digite o andar de destino.
    * *Critérios de Aceitação:* Aceita apenas inteiros de `0` (Térreo) a `10`. Caso seja inválido, exibe mensagem de erro e solicita novamente.
* **RF-002: Monitorar em Tempo Real (CLI)**
    * *Descrição:* Interface via terminal que mostra graficamente a posição exata das cabines A e B.
    * *Critérios de Aceitação:* O painel atualiza automaticamente a cada movimento das cabines.
* **RF-003: Quantidade de Passageiros antes do Embarque**
    * *Descrição:* Solicitação da quantidade de pessoas entrando na cabine para controle de carga.
* **RF-004: Alerta Máximo**
    * *Descrição:* Emissão de alerta sonoro/textual visual se a capacidade for ultrapassada.
    * *Critérios de Aceitação:* O elevador fica travado caso haja superlotação.
* **RF-005: Parada de Emergência**
    * *Descrição:* Mecanismo de segurança para interrupção ou retorno seguro. *(Fase de implementação final).*

### 2.2 Regras de Negócio (RN)

* **RN-001: Controle de Carga Máxima**
    * *Critério:* Limite fixado em **6 passageiros**. Se violado, a cabine permanece estática.
* **RN-002: Elevador Mais Próximo**
    * *Critério:* O sistema calcula a distância absoluta ($|andar\_atual - andar\_chamado|$) para enviar a cabine mais perto. Em caso de empate, o Elevador A tem prioridade.
* **RN-003: Limites do Prédio**
    * *Critério:* Movimentação restrita rigorosamente entre os andares `0` e `10`.

### 2.3 Requisitos Não Funcionais (RNF)

* **RNF-001: Interface do Sistema**
    * *Critério:* Foco na usabilidade e robustez via texto (Terminal/CLI), sem necessidade de interface gráfica complexa.
* **RNF-002: Estado dos Elevadores**
    * *Critério:* Ao encerrar/desligar o sistema, os elevadores devem retornar de forma segura à sua posição inicial (Térreo).

---

## 🚀 3. Histórico de Sprints

* **Sprint 1: Base e Validação** ➔ Limitação do prédio (0-10) e tratamento de erros de digitação (*try-except*).
* **Sprint 2: Cabines e Painel** ➔ Modelagem POO das cabines A e B e criação do painel de telemetria assíncrono em texto.
* **Sprint 3: Inteligência e Movimento** ➔ Lógica de distância absoluta e simulação de deslocamento tempo a tempo (*delay* realístico).
* **Sprint 4: Controle de Peso** ➔ Implementação da trava de segurança para o limite de 6 pessoas.
* **Sprint 5: Emergência e Reset (Em andamento)** ➔ Ajustes finais para rotinas de desligamento e evacuação rápida.

---

## 💻 4. Código Fonte (Python)

Para executar o sistema, certifique-se de ter o Python 3 instalado. Salve o código abaixo como `main.py` e execute com o comando `python main.py`.

```python
# -*- coding: utf-8 -*-
"""
LiftLogic - Sistema de Controle de Elevadores
Tema: Prédio com 11 andares (0 a 10) e 2 elevadores (A e B).
"""
import time

ANDAR_MINIMO = 0 
ANDAR_MAXIMO = 10
CAPACIDADE_MAXIMA = 6 

class Elevador:
    """Classe que representa uma cabine de elevador."""
    def __init__(self, nome):
        self.nome = nome
        self.andar_atual = 0  # Ambos começam no térreo (0)
        self.passageiros = 0 

    def verificar_e_tratar_excesso(self):
        """RN01: Impede a partida sob qualquer hipótese até que o peso seja normalizado."""
        while self.passageiros > CAPACIDADE_MAXIMA:
            print("\n🚨 " * 3)
            print(f"[BEEP! EXCESSO DE CARGA] - Elevador {self.nome}")  
            print(f"Passageiros a bordo: {self.passageiros}. Limite máximo: {CAPACIDADE_MAXIMA}.")
            print("🚨 " * 3 + "\n")
            
            sair = obter_entrada_numerica(
                f"Quantas pessoas precisam SAIR do Elevador {self.nome} para liberar o motor? ", 
                1, 
                self.passageiros
            )
            self.passageiros -= sair
            if self.passageiros <= CAPACIDADE_MAXIMA:
                print(f"✅ Peso normalizado no Elevador {self.nome}! Movimento liberado.\n")

    def mover(self, andar_destino, passageiros_iniciais):
        """Movimenta o elevador e gerencia paradas unitárias."""
        self.passageiros += passageiros_iniciais
        self.verificar_e_tratar_excesso()

        if self.andar_atual == andar_destino:
            print(f"🛗 Elevador {self.nome} já está no andar {andar_destino}.")
            return

        # Define a direção do movimento
        passo = 1 if andar_destino > self.andar_atual else -1
        
        while self.andar_atual != andar_destino:
            self.andar_atual += passo
            time.sleep(0.5)  # Simula a viagem física 
            
            print(f"\n[MOVIMENTO] Elevador {self.nome} moveu-se para o Andar {self.andar_atual}.")
            exibir_painel_sincrono()
            
            # RF04: Parada unitária para coletar novas demandas se houver
            if self.andar_atual != andar_destino:
                print(f"✨ Parada intermediária no Andar {self.andar_atual}:")
                novos = obter_entrada_numerica(
                    "Alguém vai ENTRAR neste andar? (Digite a quantidade ou 0 para ninguém): ", 
                    0, 
                    50
                )
                self.passageiros += novos
                self.verificar_e_tratar_excesso()

        print(f"✅ Elevador {self.nome} chegou ao destino: Andar {andar_destino}!")
        print(f"🚪 Portas abertas. Desembarcando todos os {self.passageiros} passageiros.\n")
        self.passageiros = 0  
        exibir_painel_sincrono()

# Instanciação das cabines
elevador_A = Elevador("A")
elevador_B = Elevador("B")

def obter_entrada_numerica(mensagem, minimo, maximo):
    """Garante robustez via try-except nas entradas do teclado."""
    while True:
        try:
            entrada = input(mensagem).strip()
            if entrada == "":
                return 0
            valor = int(entrada)
            if valor < minimo or valor > maximo:
                print(f"⚠️ Valor inválido! Deve estar entre {minimo} e {maximo}. Tente novamente.")
                continue
            return valor
        except ValueError:
            print("⚠️ Entrada inválida. Digite apenas números inteiros.")

def exibir_painel_sincrono():
    """RF02: Painel síncrono com mapeamento dos níveis de telemetria."""
    print("\n" + "=" * 50)
    print(" PAINEL TELEMETRIA EM TEMPO REAL")
    print("=" * 50)
    print(" Andar | Elevador A | Elevador B | Passageiros")
    print("-" * 50) 
    
    for andar in range(ANDAR_MAXIMO, ANDAR_MINIMO - 1, -1):
        label_andar = f"{andar:2d} (Térreo)" if andar == 0 else f"{andar:2d}        "
        
        eA = "  [A]    " if elevador_A.andar_atual == andar else "  .     "
        eB = "  [B]    " if elevador_B.andar_atual == andar else "  .     "
        
        info_pax = ""
        if elevador_A.andar_atual == andar and elevador_A.passageiros > 0:
            info_pax += f"A: {elevador_A.passageiros}pax "
        if elevador_B.andar_atual == andar and elevador_B.passageiros > 0:
            info_pax += f"B: {elevador_B.passageiros}pax "
            
        print(f" {label_andar} |   {eA}   |   {eB}   | {info_pax}")
    print("=" * 50 + "\n")

def despachar_elevador(andar_chamado):
    """RN02: Cálculo baseado na menor distância modular absoluta."""
    distancia_A = abs(elevador_A.andar_atual - andar_chamado)
    distancia_B = abs(elevador_B.andar_atual - andar_chamado) 
    
    print(f"🤖 [CÁLCULO] Distância absoluta -> Elevador A: {distancia_A} | Elevador B: {distancia_B}") 
    
    if distancia_A <= distancia_B: 
        return elevador_A
    else: 
        return elevador_B

def menu_principal():
    """Loop principal da CLI."""
    print("=" * 55)
    print(" SISTEMA INTELIGENTE COORDENADO DE ALTA PERFORMANCE")
    print(f" Configuração: Andares 0-10 | Capacidade Limite: {CAPACIDADE_MAXIMA}")
    print("=" * 55) 
    
    while True: 
        exibir_painel_sincrono()
        print("1 - Registrar Chamado (Mover Elevador)")
        print("2 - Sair do Sistema")
        
        opcao = obter_entrada_numerica("Escolha uma opção: ", 1, 2) 

        if opcao == 1: 
            andar_origem = obter_entrada_numerica(
                f"Em qual andar você está agora ({ANDAR_MINIMO}-{ANDAR_MAXIMO})? ",  
                ANDAR_MINIMO,  
                ANDAR_MAXIMO 
            )
            andar_destino = obter_entrada_numerica(
                f"Qual o seu andar de destino ({ANDAR_MINIMO}-{ANDAR_MAXIMO})? ",  
                ANDAR_MINIMO,  
                ANDAR_MAXIMO 
            )
            
            if andar_origem == andar_destino:
                print("⚠️ Erro: Andar de origem e destino não podem ser iguais.")
                continue
                
            passageiros = obter_entrada_numerica("Quantos passageiros vão entrar no início do chamado? ", 1, 50) 
            
            # Escolha da melhor cabine 
            elevador_escolhido = despachar_elevador(andar_origem) 
            print(f"\n🤖 [SISTEMA] O Elevador {elevador_escolhido.nome} foi escalado.") 
            
            # Se não estiver no andar de origem, vai buscar vazio
            if elevador_escolhido.andar_atual != andar_origem: 
                print(f"🚚 Deslocando Elevador {elevador_escolhido.nome} até o andar {andar_origem} para embarque...") 
                elevador_escolhido.mover(andar_origem, 0) 
            
            # Inicia o trajeto real
            print(f"🛫 Iniciando trajeto do andar {andar_origem} para o destino {andar_destino}...") 
            elevador_escolhido.mover(andar_destino, passageiros) 

        elif opcao == 2: 
            # RNF-002: Rotina simples de retorno ao térreo ao fechar
            print("\nRetornando elevadores ao térreo por segurança...")
            elevador_A.andar_atual = 0
            elevador_B.andar_atual = 0
            print("Encerrando o sistema predial. Até logo! 👋") 
            break 

if __name__ == "__main__":
    menu_principal()
