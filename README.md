# Sistema de Reserva de Hotéis

## Descrição

Este sistema simula reservas de hotéis e inclui funcionalidades básicas como verificar disponibilidade, fazer reservas, cancelar e modificar reservas. O sistema utiliza SQLite como banco de dados para armazenar informações sobre quartos, reservas e clientes. 

## Funcionalidades

1. **Adicionar Quarto:** Adiciona um novo quarto ao banco de dados.
2. **Verificar Disponibilidade:** Verifica a disponibilidade de quartos para um intervalo de datas.
3. **Fazer Reserva:** Faz uma reserva para um quarto disponível.
4. **Cancelar Reserva:** Cancela uma reserva existente.
5. **Modificar Reserva:** Modifica uma reserva existente com novas datas.

## Tecnologias Utilizadas

- **Linguagem:** Python
- **Banco de Dados:** SQLite
- **Interface:** Console

## Estrutura do Banco de Dados

O banco de dados `hotel_reservation.db` contém duas tabelas principais:

1. **rooms**
   - `room_id` (INTEGER, PRIMARY KEY): Identificador único do quarto.
   - `room_number` (TEXT, UNIQUE): Número do quarto.
   - `status` (TEXT): Status do quarto (disponível ou reservado).

2. **reservations**
   - `reservation_id` (INTEGER, PRIMARY KEY): Identificador único da reserva.
   - `room_id` (INTEGER): Identificador do quarto reservado.
   - `customer_name` (TEXT): Nome do cliente.
   - `start_date` (TEXT): Data de início da reserva.
   - `end_date` (TEXT): Data de término da reserva.
   - **FOREIGN KEY (room_id) REFERENCES rooms(room_id):** Relaciona a reserva ao quarto correspondente.

## Código

```python
import sqlite3

# Função para criar o banco de dados e as tabelas
def initialize_database():
    with sqlite3.connect('hotel_reservation.db') as conn:
        cursor = conn.cursor()
        cursor.execute('''CREATE TABLE IF NOT EXISTS rooms (
                            room_id INTEGER PRIMARY KEY,
                            room_number TEXT UNIQUE,
                            status TEXT)''')
        cursor.execute('''CREATE TABLE IF NOT EXISTS reservations (
                            reservation_id INTEGER PRIMARY KEY,
                            room_id INTEGER,
                            customer_name TEXT,
                            start_date TEXT,
                            end_date TEXT,
                            FOREIGN KEY (room_id) REFERENCES rooms (room_id))''')
        conn.commit()

# Função para adicionar um quarto ao banco de dados
def add_room(room_number):
    with sqlite3.connect('hotel_reservation.db') as conn:
        cursor = conn.cursor()
        cursor.execute('INSERT INTO rooms (room_number, status) VALUES (?, ?)', (room_number, 'available'))
        conn.commit()

# Função para buscar disponibilidade de quartos
def check_availability(start_date, end_date):
    with sqlite3.connect('hotel_reservation.db') as conn:
        cursor = conn.cursor()
        cursor.execute('''SELECT room_number FROM rooms WHERE status = 'available' AND room_id NOT IN (
                            SELECT room_id FROM reservations WHERE (start_date <= ? AND end_date >= ?) 
                            OR (start_date <= ? AND end_date >= ?))''', 
                            (end_date, start_date, end_date, start_date))
        available_rooms = cursor.fetchall()
        return available_rooms

# Função para fazer uma reserva
def make_reservation(room_number, customer_name, start_date, end_date):
    with sqlite3.connect('hotel_reservation.db') as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT room_id FROM rooms WHERE room_number = ? AND status = "available"', (room_number,))
        room = cursor.fetchone()
        if room:
            room_id = room[0]
            cursor.execute('INSERT INTO reservations (room_id, customer_name, start_date, end_date) VALUES (?, ?, ?, ?)',
                           (room_id, customer_name, start_date, end_date))
            cursor.execute('UPDATE rooms SET status = "reserved" WHERE room_id = ?', (room_id,))
            conn.commit()
            return f"Reserva feita com sucesso para o quarto {room_number}."
        else:
            return "Quarto não disponível."

# Função para cancelar uma reserva
def cancel_reservation(reservation_id):
    with sqlite3.connect('hotel_reservation.db') as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT room_id FROM reservations WHERE reservation_id = ?', (reservation_id,))
        room_id = cursor.fetchone()
        if room_id:
            cursor.execute('DELETE FROM reservations WHERE reservation_id = ?', (reservation_id,))
            cursor.execute('UPDATE rooms SET status = "available" WHERE room_id = ?', (room_id[0],))
            conn.commit()
            return "Reserva cancelada com sucesso."
        else:
            return "Reserva não encontrada."

# Função para modificar uma reserva
def modify_reservation(reservation_id, new_start_date, new_end_date):
    with sqlite3.connect('hotel_reservation.db') as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT room_id FROM reservations WHERE reservation_id = ?', (reservation_id,))
        room_id = cursor.fetchone()
        if room_id:
            cursor.execute('UPDATE reservations SET start_date = ?, end_date = ? WHERE reservation_id = ?',
                           (new_start_date, new_end_date, reservation_id))
            conn.commit()
            return "Reserva modificada com sucesso."
        else:
            return "Reserva não encontrada."

# Função principal para interação com o usuário
def main():
    initialize_database()
    
    while True:
        print("\nSistema de Reserva de Hotéis")
        print("1. Adicionar quarto")
        print("2. Verificar disponibilidade")
        print("3. Fazer reserva")
        print("4. Cancelar reserva")
        print("5. Modificar reserva")
        print("6. Sair")
        
        choice = input("Escolha uma opção: ")
        
        if choice == '1':
            room_number = input("Número do quarto: ")
            print(add_room(room_number))
        elif choice == '2':
            start_date = input("Data de início (YYYY-MM-DD): ")
            end_date = input("Data de término (YYYY-MM-DD): ")
            available_rooms = check_availability(start_date, end_date)
            print("Quartos disponíveis:", available_rooms)
        elif choice == '3':
            room_number = input("Número do quarto: ")
            customer_name = input("Nome do cliente: ")
            start_date = input("Data de início (YYYY-MM-DD): ")
            end_date = input("Data de término (YYYY-MM-DD): ")
            print(make_reservation(room_number, customer_name, start_date, end_date))
        elif choice == '4':
            reservation_id = int(input("ID da reserva: "))
            print(cancel_reservation(reservation_id))
        elif choice == '5':
            reservation_id = int(input("ID da reserva: "))
            new_start_date = input("Nova data de início (YYYY-MM-DD): ")
            new_end_date = input("Nova data de término (YYYY-MM-DD): ")
            print(modify_reservation(reservation_id, new_start_date, new_end_date))
        elif choice == '6':
            break
        else:
            print("Opção inválida. Tente novamente.")

if __name__ == "__main__":
    main()
