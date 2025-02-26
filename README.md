
# Programación Orientada a Objetos - UNAL

# Ana María Amaya Gómez

## Reto 7: Modificación del restaurante
1.Agregue la estructura de datos adecuada para gestionar múltiples pedidos (tal vez una cola FIFO)
2.Defina una tupla con nombre en algún lugar del menú, p. para definir un conjunto de elementos.
3.Cree una interfaz en la clase de pedido, para crear un nuevo menú, agregue las funciones para agregar, actualizar y eliminar elementos. Todos los menús deben almacenarse como archivos JSON. (use dictados para esta tarea).

#Código base
```python
class MenuItem:
    def __init__(self, name: str, price: float, type: str):
        self._name = name
        self._price = price
        self._type = type

    def get_name(self):
        return self._name

    def set_name(self, name: str):
        self._name = name

    def get_price(self):
        return self._price

    def set_price(self, price: float):
        self._price = price

    def get_type(self):
        return self._type

    def set_type(self, type: str):
        self._type = type

    def total_price(self, quantity) -> float:
        return quantity * self._price

class Appetizer(MenuItem):
    def __init__(self, name: str, price: float, type: str, serving: str):
        super().__init__(name, price, type)
        self._serving = serving

    def get_serving(self):
        return self._serving

    def set_serving(self, serving: str):
        self._serving = serving

class Beverage(MenuItem):
    def __init__(self, name: str, price: float, type: str, size: str):
        super().__init__(name, price, type)
        self._size = size

    def get_size(self):
        return self._size

    def set_size(self, size: str):
        self._size = size

class Dessert(MenuItem):
    def __init__(self, name: str, price: float, type: str, flavor: str):
        super().__init__(name, price, type)
        self._flavor = flavor

    def get_flavor(self):
        return self._flavor

    def set_flavor(self, flavor: str):
        self._flavor = flavor

class MainCourse(MenuItem):
    def __init__(self, name: str, price: float, type: str, protein: str, vegetable: str, starch: str):
        super().__init__(name, price, type)
        self._protein = protein
        self._vegetable = vegetable
        self._starch = starch

    def get_protein(self):
        return self._protein

    def set_protein(self, protein: str):
        self._protein = protein

    def get_vegetable(self):
        return self._vegetable

    def set_vegetable(self, vegetable: str):
        self._vegetable = vegetable

    def get_starch(self):
        return self._starch

    def set_starch(self, starch: str):
        self._starch = starch

class Order:
    def __init__(self):
        self.items = []

    def add_item(self, item: MenuItem, quantity: int = 1):
        self.items.append((item, quantity))

    def calculate_total(self) -> float:
        total = sum(item.total_price(quantity) for item, quantity in self.items)
        return total

    def apply_discount(self, discount: float) -> float:
        total = self.calculate_total()
        return total * (1 - discount)

    def display_order(self):
        for item, quantity in self.items:
            print(f'{quantity} x {item.get_name()} - {item.total_price(quantity):.2f} (unidad: {item.get_price():.2f})')

        total = self.calculate_total()
        print(f'Total: {total:.2f}')

    def pay(self, medio_pago: "MedioPago"):
        total = self.calculate_total()
        medio_pago.pagar(total)


class MedioPago:
    def __init__(self):
        pass

    def pagar(self, monto):
        pass

class Tarjeta(MedioPago):
    def __init__(self, numero, cvv):
        super().__init__()
        self.numero = numero
        self.cvv = cvv

    def pagar(self, monto):
        print(f"Pagando {monto:.2f} con tarjeta {self.numero[-4:]}")

class Efectivo(MedioPago):
    def __init__(self, monto_entregado):
        super().__init__()
        self.monto_entregado = monto_entregado

    def pagar(self, monto):
        if self.monto_entregado >= monto:
            print(f"Pago realizado en efectivo. Cambio: {self.monto_entregado - monto:.2f}")
        else:
            print(f"Fondos insuficientes. Faltan {monto - self.monto_entregado:.2f} para completar el pago.")

bebida1 = Beverage("agua", 2.75, "sin gas", "250 ml")
entrada1 = Appetizer("Mini empanadas", 13.50, "Carne", "Grupal")
plato1 = MainCourse("Plato del día", 24.00, "corriente", "carne", "ensalada", "arroz")

order = Order()
order.add_item(bebida1, 2)
order.add_item(entrada1, 1)
order.add_item(plato1, 1)

print("Orden antes del descuento:")
order.display_order()

descuento = 0.10
total_con_descuento = order.apply_discount(descuento)
print(f"\nTotal con {descuento * 100}% de descuento: {total_con_descuento:.2f}")

print("\nRealizando el pago:")
tarjeta = Tarjeta("1234567890123456", "123")
efectivo = Efectivo(50.00)
order.pay(tarjeta)  
order.pay(efectivo)
```

##1 Multiples pedidos
Para lograr un sistema de multiples pedidos utilizaremos la clase deque de la biblioteca estándar de Python collections para implementar una cola FIFO (First In, First Out). La cola FIFO nos permitirá gestionar los pedidos en el orden en que se recibieron.

```python
from collections import deque

class MenuItem:
    def __init__(self, name: str, price: float, type: str):
        self._name = name
        self._price = price
        self._type = type

    def get_name(self):
        return self._name

    def set_name(self, name: str):
        self._name = name

    def get_price(self):
        return self._price

    def set_price(self, price: float):
        self._price = price

    def get_type(self):
        return self._type

    def set_type(self, type: str):
        self._type = type

    def total_price(self, quantity) -> float:
        return quantity * self._price

class Appetizer(MenuItem):
    def __init__(self, name: str, price: float, type: str, serving: str):
        super().__init__(name, price, type)
        self._serving = serving

    def get_serving(self):
        return self._serving

    def set_serving(self, serving: str):
        self._serving = serving

class Beverage(MenuItem):
    def __init__(self, name: str, price: float, type: str, size: str):
        super().__init__(name, price, type)
        self._size = size

    def get_size(self):
        return self._size

    def set_size(self, size: str):
        self._size = size

class Dessert(MenuItem):
    def __init__(self, name: str, price: float, type: str, flavor: str):
        super().__init__(name, price, type)
        self._flavor = flavor

    def get_flavor(self):
        return self._flavor

    def set_flavor(self, flavor: str):
        self._flavor = flavor

class MainCourse(MenuItem):
    def __init__(self, name: str, price: float, type: str, protein: str, vegetable: str, starch: str):
        super().__init__(name, price, type)
        self._protein = protein
        self._vegetable = vegetable
        self._starch = starch

    def get_protein(self):
        return self._protein

    def set_protein(self, protein: str):
        self._protein = protein

    def get_vegetable(self):
        return self._vegetable

    def set_vegetable(self, vegetable: str):
        self._vegetable = vegetable

    def get_starch(self):
        return self._starch

    def set_starch(self, starch: str):
        self._starch = starch

class Order:
    def __init__(self):
        self.items = []

    def add_item(self, item: MenuItem, quantity: int = 1):
        self.items.append((item, quantity))

    def calculate_total(self) -> float:
        total = sum(item.total_price(quantity) for item, quantity in self.items)
        return total

    def apply_discount(self, discount: float) -> float:
        total = self.calculate_total()
        return total * (1 - discount)

    def display_order(self):
        for item, quantity in self.items:
            print(f'{quantity} x {item.get_name()} - {item.total_price(quantity):.2f} (unidad: {item.get_price():.2f})')

        total = self.calculate_total()
        print(f'Total: {total:.2f}')

    def pay(self, medio_pago: "MedioPago"):
        total = self.calculate_total()
        medio_pago.pagar(total)

class MedioPago:
    def __init__(self):
        pass

    def pagar(self, monto):
        pass

class Tarjeta(MedioPago):
    def __init__(self, numero, cvv):
        super().__init__()
        self.numero = numero
        self.cvv = cvv

    def pagar(self, monto):
        print(f"Pagando {monto:.2f} con tarjeta {self.numero[-4:]}")

class Efectivo(MedioPago):
    def __init__(self, monto_entregado):
        super().__init__()
        self.monto_entregado = monto_entregado

    def pagar(self, monto):
        if self.monto_entregado >= monto:
            print(f"Pago realizado en efectivo. Cambio: {self.monto_entregado - monto:.2f}")
        else:
            print(f"Fondos insuficientes. Faltan {monto - self.monto_entregado:.2f} para completar el pago.")

class MenuItems:
    def __init__(self):
        self.items = []

    def add_item(self, item: MenuItem):
        self.items.append(item)

    def __iter__(self):
        return iter(self.items)

class OrdersQueue:
    def __init__(self):
        self.orders = deque()

    def add_order(self, order: Order):
        self.orders.append(order)

    def get_next_order(self):
        return self.orders.popleft() if self.orders else None


bebida1 = Beverage("agua", 2.75, "sin gas", "250 ml")
entrada1 = Appetizer("Mini empanadas", 13.50, "Carne", "Grupal")
plato1 = MainCourse("Plato del día", 24.00, "corriente", "carne", "ensalada", "arroz")

menu_items = MenuItems()
menu_items.add_item(bebida1)
menu_items.add_item(entrada1)
menu_items.add_item(plato1)


for item in menu_items:
    print(f'Nombre: {item.get_name()}, Precio: {item.get_price():.2f}, Tipo: {item.get_type()}')


for item in menu_items:
    if isinstance(item, Beverage):
        print(f'Tamaño de la bebida: {item.get_size()}')
    elif isinstance(item, Appetizer):
        print(f'Tipo de porción: {item.get_serving()}')
    elif isinstance(item, Dessert):
        print(f'Sabor del postre: {item.get_flavor()}')
    elif isinstance(item, MainCourse):
        print(f'Proteína: {item.get_protein()}, Vegetales: {item.get_vegetable()}, Carbohidrato: {item.get_starch()}')


order1 = Order()
order1.add_item(bebida1, 2)
order1.add_item(entrada1, 1)
order1.add_item(plato1, 1)


order2 = Order()
order2.add_item(bebida1, 1)
order2.add_item(entrada1, 2)


orders_queue = OrdersQueue()
orders_queue.add_order(order1)
orders_queue.add_order(order2)


print("\nProcesando órdenes:")
while orders_queue.orders:
    current_order = orders_queue.get_next_order()
    current_order.display_order()
    print()

```
Resultado 

```python
Nombre: agua, Precio: 2.75, Tipo: sin gas
Nombre: Mini empanadas, Precio: 13.50, Tipo: Carne
Nombre: Plato del día, Precio: 24.00, Tipo: corriente
Tamaño de la bebida: 250 ml
Tipo de porción: Grupal
Proteína: carne, Vegetales: ensalada, Carbohidrato: arroz

Procesando órdenes:
2 x agua - 5.50 (unidad: 2.75)
1 x Mini empanadas - 13.50 (unidad: 13.50)
1 x Plato del día - 24.00 (unidad: 24.00)
Total: 43.00

1 x agua - 2.75 (unidad: 2.75)
2 x Mini empanadas - 27.00 (unidad: 13.50)
Total: 29.75
```

#2.Conjunto de elementos
para esto vamos a utilizar namedtuple de la biblioteca estándar collections para definir una tupla con nombre que nos permitirá representar un conjunto de elementos.

```python
from collections import deque, namedtuple

# Definimos la tupla con nombre
MenuItemDetails = namedtuple('MenuItemDetails', ['name', 'price', 'type'])

class MenuItem:
    def __init__(self, name: str, price: float, type: str):
        self._name = name
        self._price = price
        self._type = type

    def get_name(self):
        return self._name

    def set_name(self, name: str):
        self._name = name

    def get_price(self):
        return self._price

    def set_price(self, price: float):
        self._price = price

    def get_type(self):
        return self._type

    def set_type(self, type: str):
        self._type = type

    def total_price(self, quantity) -> float:
        return quantity * self._price

class Appetizer(MenuItem):
    def __init__(self, name: str, price: float, type: str, serving: str):
        super().__init__(name, price, type)
        self._serving = serving

    def get_serving(self):
        return self._serving

    def set_serving(self, serving: str):
        self._serving = serving

class Beverage(MenuItem):
    def __init__(self, name: str, price: float, type: str, size: str):
        super().__init__(name, price, type)
        self._size = size

    def get_size(self):
        return self._size

    def set_size(self, size: str):
        self._size = size

class Dessert(MenuItem):
    def __init__(self, name: str, price: float, type: str, flavor: str):
        super().__init__(name, price, type)
        self._flavor = flavor

    def get_flavor(self):
        return self._flavor

    def set_flavor(self, flavor: str):
        self._flavor = flavor

class MainCourse(MenuItem):
    def __init__(self, name: str, price: float, type: str, protein: str, vegetable: str, starch: str):
        super().__init__(name, price, type)
        self._protein = protein
        self._vegetable = vegetable
        self._starch = starch

    def get_protein(self):
        return self._protein

    def set_protein(self, protein: str):
        self._protein = protein

    def get_vegetable(self):
        return self._vegetable

    def set_vegetable(self, vegetable: str):
        self._vegetable = vegetable

    def get_starch(self):
        return self._starch

    def set_starch(self, starch: str):
        self._starch = starch

class Order:
    def __init__(self):
        self.items = []

    def add_item(self, item: MenuItem, quantity: int = 1):
        self.items.append((item, quantity))

    def calculate_total(self) -> float:
        total = sum(item.total_price(quantity) for item, quantity in self.items)
        return total

    def apply_discount(self, discount: float) -> float:
        total = self.calculate_total()
        return total * (1 - discount)

    def display_order(self):
        for item, quantity in self.items:
            print(f'{quantity} x {item.get_name()} - {item.total_price(quantity):.2f} (unidad: {item.get_price():.2f})')

        total = self.calculate_total()
        print(f'Total: {total:.2f}')

    def pay(self, medio_pago: "MedioPago"):
        total = self.calculate_total()
        medio_pago.pagar(total)

class MedioPago:
    def __init__(self):
        pass

    def pagar(self, monto):
        pass

class Tarjeta(MedioPago):
    def __init__(self, numero, cvv):
        super().__init__()
        self.numero = numero
        self.cvv = cvv

    def pagar(self, monto):
        print(f"Pagando {monto:.2f} con tarjeta {self.numero[-4:]}")

class Efectivo(MedioPago):
    def __init__(self, monto_entregado):
        super().__init__()
        self.monto_entregado = monto_entregado

    def pagar(self, monto):
        if self.monto_entregado >= monto:
            print(f"Pago realizado en efectivo. Cambio: {self.monto_entregado - monto:.2f}")
        else:
            print(f"Fondos insuficientes. Faltan {monto - self.monto_entregado:.2f} para completar el pago.")

class MenuItems:
    def __init__(self):
        self.items = []

    def add_item(self, item: MenuItem):
        self.items.append(item)

    def __iter__(self):
        return iter(self.items)

class OrdersQueue:
    def __init__(self):
        self.orders = deque()

    def add_order(self, order: Order):
        self.orders.append(order)

    def get_next_order(self):
        return self.orders.popleft() if self.orders else None

# Instancia de los ítems del menú
bebida1 = Beverage("agua", 2.75, "sin gas", "250 ml")
entrada1 = Appetizer("Mini empanadas", 13.50, "Carne", "Grupal")
plato1 = MainCourse("Plato del día", 24.00, "corriente", "carne", "ensalada", "arroz")

# Añadiendo ítems al menú
menu_items = MenuItems()
menu_items.add_item(bebida1)
menu_items.add_item(entrada1)
menu_items.add_item(plato1)

# Definiendo un conjunto de elementos utilizando namedtuple
combo1 = MenuItemDetails('Combo Familiar', 30.00, 'combo')
print(f'Nombre: {combo1.name}, Precio: {combo1.price}, Tipo: {combo1.type}')

# Ejemplo de uso iterando sobre los ítems del menú
for item in menu_items:
    print(f'Nombre: {item.get_name()}, Precio: {item.get_price():.2f}, Tipo: {item.get_type()}')

# También puedes acceder a los atributos específicos de los elementos
for item in menu_items:
    if isinstance(item, Beverage):
        print(f'Tamaño de la bebida: {item.get_size()}')
    elif isinstance(item, Appetizer):
        print(f'Tipo de porción: {item.get_serving()}')
    elif isinstance(item, Dessert):
        print(f'Sabor del postre: {item.get_flavor()}')
    elif isinstance(item, MainCourse):
        print(f'Proteína: {item.get_protein()}, Vegetales: {item.get_vegetable()}, Carbohidrato: {item.get_starch()}')

# Creación y visualización de una orden
order1 = Order()
order1.add_item(bebida1, 2)
order1.add_item(entrada1, 1)
order1.add_item(plato1, 1)

# Creación de otra orden
order2 = Order()
order2.add_item(bebida1, 1)
order2.add_item(entrada1, 2)

# Añadiendo órdenes a la cola de pedidos
orders_queue = OrdersQueue()
orders_queue.add_order(order1)
orders_queue.add_order(order2)

# Procesando órdenes en orden FIFO
print("\nProcesando órdenes:")
while orders_queue.orders:
    current_order = orders_queue.get_next_order()
    current_order.display_order()
    print()

```

Resultado 

```python
Nombre: Combo Familiar, Precio: 30.0, Tipo: combo
Nombre: agua, Precio: 2.75, Tipo: sin gas
Nombre: Mini empanadas, Precio: 13.50, Tipo: Carne
Nombre: Plato del día, Precio: 24.00, Tipo: corriente
Tamaño de la bebida: 250 ml
Tipo de porción: Grupal
Proteína: carne, Vegetales: ensalada, Carbohidrato: arroz

Procesando órdenes:
2 x agua - 5.50 (unidad: 2.75)
1 x Mini empanadas - 13.50 (unidad: 13.50)
1 x Plato del día - 24.00 (unidad: 24.00)
Total: 43.00

1 x agua - 2.75 (unidad: 2.75)
2 x Mini empanadas - 27.00 (unidad: 13.50)
Total: 29.75
```

##3. json

Vamos a implementar una interfaz en la clase Order para manejar menús y asegurarnos de que los menús se puedan agregar, actualizar y eliminar. Además, utilizaremos el módulo json para guardar y cargar los menús desde archivos JSON.

```python
import json
from collections import deque, namedtuple


MenuItemDetails = namedtuple('MenuItemDetails', ['name', 'price', 'type'])

class MenuItem:
    def __init__(self, name: str, price: float, type: str):
        self._name = name
        self._price = price
        self._type = type

    def get_name(self):
        return self._name

    def set_name(self, name: str):
        self._name = name

    def get_price(self):
        return self._price

    def set_price(self, price: float):
        self._price = price

    def get_type(self):
        return self._type

    def set_type(self, type: str):
        self._type = type

    def total_price(self, quantity) -> float:
        return quantity * self._price

    def to_dict(self):
        return {
            'name': self._name,
            'price': self._price,
            'type': self._type
        }

class Appetizer(MenuItem):
    def __init__(self, name: str, price: float, type: str, serving: str):
        super().__init__(name, price, type)
        self._serving = serving

    def get_serving(self):
        return self._serving

    def set_serving(self, serving: str):
        self._serving = serving

    def to_dict(self):
        data = super().to_dict()
        data['serving'] = self._serving
        return data

class Beverage(MenuItem):
    def __init__(self, name: str, price: float, type: str, size: str):
        super().__init__(name, price, type)
        self._size = size

    def get_size(self):
        return self._size

    def set_size(self, size: str):
        self._size = size

    def to_dict(self):
        data = super().to_dict()
        data['size'] = self._size
        return data

class Dessert(MenuItem):
    def __init__(self, name: str, price: float, type: str, flavor: str):
        super().__init__(name, price, type)
        self._flavor = flavor

    def get_flavor(self):
        return self._flavor

    def set_flavor(self, flavor: str):
        self._flavor = flavor

    def to_dict(self):
        data = super().to_dict()
        data['flavor'] = self._flavor
        return data

class MainCourse(MenuItem):
    def __init__(self, name: str, price: float, type: str, protein: str, vegetable: str, starch: str):
        super().__init__(name, price, type)
        self._protein = protein
        self._vegetable = vegetable
        self._starch = starch

    def get_protein(self):
        return self._protein

    def set_protein(self, protein: str):
        self._protein = protein

    def get_vegetable(self):
        return self._vegetable

    def set_vegetable(self, vegetable: str):
        self._vegetable = vegetable

    def get_starch(self):
        return self._starch

    def set_starch(self, starch: str):
        self._starch = starch

    def to_dict(self):
        data = super().to_dict()
        data['protein'] = self._protein
        data['vegetable'] = self._vegetable
        data['starch'] = self._starch
        return data

class Order:
    def __init__(self):
        self.items = []

    def add_item(self, item: MenuItem, quantity: int = 1):
        self.items.append((item, quantity))

    def calculate_total(self) -> float:
        total = sum(item.total_price(quantity) for item, quantity in self.items)
        return total

    def apply_discount(self, discount: float) -> float:
        total = self.calculate_total()
        return total * (1 - discount)

    def display_order(self):
        for item, quantity in self.items:
            print(f'{quantity} x {item.get_name()} - {item.total_price(quantity):.2f} (unidad: {item.get_price():.2f})')

        total = self.calculate_total()
        print(f'Total: {total:.2f}')

    def pay(self, medio_pago: "MedioPago"):
        total = self.calculate_total()
        medio_pago.pagar(total)

    # Interfaz para manejar menús
    @staticmethod
    def create_menu(menu_file: str):
        with open(menu_file, 'w') as file:
            json.dump({}, file)

    @staticmethod
    def add_menu_item(menu_file: str, item: MenuItem):
        with open(menu_file, 'r') as file:
            menu = json.load(file)

        menu[item.get_name()] = item.to_dict()

        with open(menu_file, 'w') as file:
            json.dump(menu, file, indent=4)

    @staticmethod
    def update_menu_item(menu_file: str, item: MenuItem):
        with open(menu_file, 'r') as file:
            menu = json.load(file)

        menu[item.get_name()] = item.to_dict()

        with open(menu_file, 'w') as file:
            json.dump(menu, file, indent=4)

    @staticmethod
    def delete_menu_item(menu_file: str, item_name: str):
        with open(menu_file, 'r') as file:
            menu = json.load(file)

        if item_name in menu:
            del menu[item_name]

        with open(menu_file, 'w') as file:
            json.dump(menu, file, indent=4)

    @staticmethod
    def load_menu(menu_file: str):
        with open(menu_file, 'r') as file:
            menu = json.load(file)
        return menu

class MedioPago:
    def __init__(self):
        pass

    def pagar(self, monto):
        pass

class Tarjeta(MedioPago):
    def __init__(self, numero, cvv):
        super().__init__()
        self.numero = numero
        self.cvv = cvv

    def pagar(self, monto):
        print(f"Pagando {monto:.2f} con tarjeta {self.numero[-4:]}")

class Efectivo(MedioPago):
    def __init__(self, monto_entregado):
        super().__init__()
        self.monto_entregado = monto_entregado

    def pagar(self, monto):
        if self.monto_entregado >= monto:
            print(f"Pago realizado en efectivo. Cambio: {self.monto_entregado - monto:.2f}")
        else:
            print(f"Fondos insuficientes. Faltan {monto - self.monto_entregado:.2f} para completar el pago.")

