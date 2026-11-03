// 1. Responsabilidad: Validar Stock (Principio S)
class ValidadorStock {
  validar(productos) {
    for (let item of productos) {
      if (item.cantidad > item.stockDisponible) {
        throw new Error(`Stock insuficiente para el producto: ${item.nombre}`);
      }
    }
  }
}

// 2. Responsabilidad: Cálculo de Totales con ISV (Principio S)
class CalculadoraPedido {
  calcularTotales(productos, tasaISV = 0.15) {
    const subtotal = productos.reduce((sum, item) => sum + (item.precio * item.cantidad), 0);
    const isv = subtotal * tasaISV;
    const total = subtotal + isv;
    return { subtotal, isv, total };
  }
}

// 3. Responsabilidad: Formato e Impresión de Ticket (Principio S)
class ImpresorTicket {
  imprimir(cliente, productos, totales) {
    console.log("=== TICKET DE COMPRA ===");
    console.log(`Cliente: ${cliente.nombre}`);
    productos.forEach(item => {
      console.log(`${item.nombre} x${item.cantidad} - $${item.precio * item.cantidad}`);
    });
    console.log(`Subtotal: $${totales.subtotal}`);
    console.log(`ISV: $${totales.isv}`);
    console.log(`Total: $${totales.total}`);
    console.log("=======================");
  }
}

// 4. Principio D (Abstracciones e Inyección de Dependencias)
class PedidoRepository {
  guardar(cliente, totales) {
    throw new Error("Método abstracto debe ser implementado");
  }
}

class Notificador {
  notificar(cliente, mensaje) {
    throw new Error("Método abstracto debe ser implementado");
  }
}

class MySQLPedidoRepository extends PedidoRepository {
  constructor(dbConnection) {
    super();
    this.db = dbConnection;
  }
  guardar(cliente, totales) {
    const query = "INSERT INTO pedidos (cliente_id, subtotal, isv, total) VALUES (?, ?, ?, ?)";
    this.db.query(query, [cliente.id, totales.subtotal, totales.isv, totales.total]);
  }
}

class WhatsAppNotificador extends Notificador {
  constructor(whatsappClient) {
    super();
    this.whatsapp = whatsappClient;
  }
  notificar(cliente, mensaje) {
    this.whatsapp.enviarMensaje(cliente.telefono, mensaje);
  }
}

// Servicio principal orquestador con Inyección de Dependencias (Principios S y D)
class ServicioPedido {
  constructor(validadorStock, calculadora, impresor, repositorio, notificador) {
    this.validadorStock = validadorStock;
    this.calculadora = calculadora;
    this.impresor = impresor;
    this.repositorio = repositorio; // Inyectado (D)
    this.notificador = notificador; // Inyectado (D)
  }

  procesarPedido(cliente, productos) {
    this.validadorStock.validar(productos);
    const totales = this.calculadora.calcularTotales(productos);
    this.repositorio.guardar(cliente, totales);
    this.impresor.imprimir(cliente, productos, totales);
    this.notificador.notificar(cliente, `Hola ${cliente.nombre}, tu pedido por un total de $${totales.total} fue procesado.`);
    return { exito: true, total: totales.total };
  }
}