# Hotelku
import streamlit as st
import datetime
import json
import pandas as pd
import requests
import streamlit as st
from streamlit_lottie import st_lottie

class TreeNode:
    def __init__(self, room_number, room_type, floor, status):
        self.room_number = room_number
        self.room_type = room_type
        self.floor = floor
        self.status = status
        self.left = None
        self.right = None


class HotelReservationSystem:
    def __init__(self):
        self.root = None
        

    def insert_room(self, room_number, room_type, floor, status):
        self.root = self._insert_room(self.root, room_number, room_type, floor, status)

    def _insert_room(self, node, room_number, room_type, floor, status):
        if node is None:
            return TreeNode(room_number, room_type, floor, status)
        if room_number < node.room_number:
            node.left = self._insert_room(node.left, room_number, room_type, floor, status)
        elif room_number > node.room_number:
            node.right = self._insert_room(node.right, room_number, room_type, floor, status)
        else:
            node.status = status  # Mengubah status kamar saat nomor kamar sudah ada
        return node

    def delete_room(self, room_number):
        self.root = self._delete_room(self.root, room_number)

    def _delete_room(self, node, room_number):
        if node is None:
            return node
        if room_number < node.room_number:
            node.left = self._delete_room(node.left, room_number)
        elif room_number > node.room_number:
            node.right = self._delete_room(node.right, room_number)
        else:
            if node.left is None and node.right is None:
                return None
            elif node.left is None:
                return node.right
            elif node.right is None:
                return node.left
            else:
                successor = self._find_min_node(node.right)
                node.room_number = successor.room_number
                node.room_type = successor.room_type
                node.floor = successor.floor
                node.status = successor.status
                node.right = self._delete_room(node.right, successor.room_number)
        return node

    def _find_min_node(self, node):
        current = node
        while current.left is not None:
            current = current.left
        return current

    def display_status(self):
        self._display_status(self.root)

    def _display_status(self, node):
        if node is not None:
            self._display_status(node.left)
            st.write("Room Number:", node.room_number)
            st.write("Room Type:", node.room_type)
            st.write("Floor:", node.floor)
            st.write("Status:", node.status)
            st.write("")  # Empty line for separation
            self._display_status(node.right)

    def get_room_data(self):
        room_data = []
        self._get_room_data(self.root, room_data)
        return room_data

    def _get_room_data(self, node, room_data):
        if node is not None:
            self._get_room_data(node.left, room_data)
            room_data.append(
                {
                    "No Kamar": node.room_number,
                    "Tipe Kamar": node.room_type,
                    "Lantai": node.floor,
                    "Status": node.status,
                }
            )
            self._get_room_data(node.right, room_data)

    def save_transactions(self, transactions):
        with open("transactions.json", "w") as file:
            json.dump(transactions, file)

    def load_transactions(self):
        try:
            with open("transactions.json", "r") as file:
                transactions = json.load(file)
            return transactions
        except FileNotFoundError:
            return []

    def get_transaction_data(self, transactions):
        transaction_data = []
        for transaction in transactions:
            check_in = datetime.datetime.strptime(transaction["Tanggal Check-in"], "%Y-%m-%d").date()
            check_out = datetime.datetime.strptime(transaction["Tanggal Check-out"], "%Y-%m-%d").date()
            num_days = (check_out - check_in).days
            bayar=number(transaction["Harga Bayar"])
            total_payment = num_days * bayar
            transaction["Harga Bayar"] = f"Rp{total_payment:,.2f}"
            transaction_data.append(transaction)
        return transaction_data
    
    def update_room_status(self, room_number, status):
        self._update_room_status(self.root, room_number, status)
    def _update_room_status(self, node, room_number, status):
        if node is not None:
            if node.room_number == room_number:
                node.status = status
            elif room_number < node.room_number:
                self._update_room_status(node.left, room_number, status)
            else:
                self._update_room_status(node.right, room_number, status)



def load_lottieurl(url: str):
    r = requests.get(url)
    if r.status_code != 200:
        return None
    return r.json()

st.title("Hotel Reservation System")
def main_menu():
    hotel_reservation_system = HotelReservationSystem()
    transactions = hotel_reservation_system.load_transactions()
    load_rooms(hotel_reservation_system)    
    lottie_url_hello = "https://assets5.lottiefiles.com/private_files/lf30_FJT2Ue.json"
    lottie_hello = load_lottieurl(lottie_url_hello)
    st_lottie(lottie_hello,loop=True,key="hello") 
    choice = st.sidebar.selectbox("Menu Utama", ("Transaksi Reservasi","Manajemen Kamar"))

    if choice == "Manajemen Kamar":
        room_menu(hotel_reservation_system)
    elif choice == "Transaksi Reservasi":
        transaction_menu(hotel_reservation_system, transactions)


    hotel_reservation_system.save_transactions(transactions)
    save_rooms(hotel_reservation_system)

def transaction_menu(hotel_reservation_system, transactions):
    choice = st.sidebar.radio(
        "Menu Transaksi Reservasi", ("Transaksi", "Riwayat Transaksi", "Pendapatan Hotel")
    )
    

    if choice == "Transaksi":
        st.subheader("Transaksi")
        customer_name = st.text_input("Nama Pelanggan")
        admin_name = st.text_input("Nama Admin")
        room_number = st.text_input("Nomor Kamar yang Dipesan")
        room_type = st.selectbox("Tipe Kamar yang Dipesan",("---","Deluxe","Family","Royal","Singel"))
        check_in = st.date_input("Tanggal Check-in", datetime.date.today())
        check_out = st.date_input("Tanggal Check-out")
        payment=0
        if room_type=="Deluxe":
            payment=250000
        elif room_type=="Family":
            payment=300000
        elif room_type=="Royal":
            payment=350000
        elif room_type=="Singel":
            payment=200000
        num_days = (check_out - check_in).days
        payemnts = num_days * payment

        if st.button("Tambahkan"):
            transaction = {
                "Nama Pelanggan": customer_name,
                "Nama Admin": admin_name,
                "Nomor Kamar": room_number,
                "Tipe Kamar": room_type,
                "Tanggal Check-in": str(check_in),
                "Tanggal Check-out": str(check_out),
                "Harga Bayar": f"Rp{payemnts:,.2f}",
            }
            transactions.append(transaction)
            st.success("Data transaksi telah ditambahkan.")

            hotel_reservation_system.update_room_status(room_number, "Booking")

    elif choice == "Riwayat Transaksi":
        st.subheader("Riwayat Transaksi")
        sort_by = "Tanggal Check-in"

        transactions.sort(key=lambda x: x[sort_by])

        transactions_df = pd.DataFrame(transactions)
        st.table(transactions_df)
        st.subheader("Hapus Data Transaksi")
        transaction_idx = st.number_input("Indeks Transaksi yang akan dihapus", min_value=0, max_value=len(transactions)-1, value=0, step=1)

        if st.button("Hapus"):
            if transaction_idx < len(transactions):
                # Retrieve the room number associated with the deleted transaction
                room_number = transactions[transaction_idx]["Nomor Kamar"]

                # Delete the transaction
                del transactions[transaction_idx]
                st.success("Data transaksi telah dihapus.")

                # Update the room status to "Tersedia"
                hotel_reservation_system.update_room_status(room_number, "Tersedia")
            else:
                st.error("Indeks transaksi tidak valid.")

    elif choice == "Pendapatan Hotel":
        st.subheader("Pendapatan Hotel")
        st.write("Jumlah Room yang Telah Di-Reservasi:")
        st.info(len(transactions))
        # Mengubah format harga menjadi angka
        def parse_price(price):
            return float(price.replace("Rp", "").replace(",", ""))

        # Mengambil nilai harga dari setiap transaksi dan menjumlahkannya
        total_payment = sum(parse_price(transaction["Harga Bayar"]) for transaction in transactions)

        # Menampilkan total pembayaran
        st.write("Total Pembayaran:")
        st.info("Rp{:.2f}".format(total_payment))

   
        

def room_menu(hotel_reservation_system):
    choice = st.sidebar.radio("Menu Manajemen Kamar", ("Input Data Kamar", "Data Kamar"))

    if choice == "Input Data Kamar":

        st.subheader("Input Data Kamar")
        room_number = st.text_input("Nomor Kamar")
        room_type = st.text_input("Tipe Kamar")
        floor = st.text_input("Lantai Kamar")
        status = st.selectbox("Status Kamar", ("Tersedia", "Booking"))

        if st.button("Tambahkan"):
            hotel_reservation_system.insert_room(room_number, room_type, floor, status)
            st.success("Data kamar telah ditambahkan.")

    elif choice == "Data Kamar":
        st.subheader("Status Kamar")
        st.write("Jumlah Room :")
    
        # Mengambil data kamar
        room_data = hotel_reservation_system.get_room_data()

        # Menghitung jumlah data kamar
        jumlah_data_kamar = len(room_data)

        # Menampilkan jumlah data kamar
        st.info(jumlah_data_kamar)       
        
        sort_options = ("Tipe Kamar", "Nomor Kamar", "Lantai Kamar")
        sort_by = st.selectbox("Urutkan berdasarkan", sort_options)

        room_data = hotel_reservation_system.get_room_data()

        if sort_by == "Tipe Kamar":
            room_data.sort(key=lambda x: x["Tipe Kamar"])
        elif sort_by == "Nomor Kamar":
            room_data.sort(key=lambda x: x["No Kamar"])
        elif sort_by == "Lantai Kamar":
            room_data.sort(key=lambda x: x["Lantai"])

        room_df = pd.DataFrame(room_data)
        st.table(room_df)
        st.subheader("Hapus Data Kamar")
        room_number = st.text_input("Nomor Kamar yang akan dihapus")

        if st.button("Hapus"):
            hotel_reservation_system.delete_room(room_number)
            st.success("Data kamar telah dihapus.")
     




def save_rooms(hotel_reservation_system):
    rooms = []
    _get_rooms_data(hotel_reservation_system.root, rooms)
    with open("rooms.json", "w") as file:
        json.dump(rooms, file)


def load_rooms(hotel_reservation_system):
    try:
        with open("rooms.json", "r") as file:
            rooms = json.load(file)
        for room in rooms:
            hotel_reservation_system.insert_room(
                room["No Kamar"], room["Tipe Kamar"], room["Lantai"], room["Status"]
            )
    except FileNotFoundError:
        pass


def _get_rooms_data(node, rooms):
    if node is not None:
        _get_rooms_data(node.left, rooms)
        room_data = {
            "No Kamar": node.room_number,
            "Tipe Kamar": node.room_type,
            "Lantai": node.floor,
            "Status": node.status,
        }
        rooms.append(room_data)
        _get_rooms_data(node.right, rooms)


if __name__ == "__main__":
    st.sidebar.image("admin.png",width=70)
    st.sidebar.header("HELLO ADMIN")
    main_menu()

