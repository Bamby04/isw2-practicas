class Pedido {
  constructor(dbConnection, whatsappClient) {
    this.db = dbConnection;
    this.whatsapp = whatsappClient;
  }

  procesarPedido(cliente, productos) {
    console.log("Iniciando procesamiento de pedido...");

    // 1. Validar Stock
    for (let item of productos) {
      if (item.cantidad > item.stockDisponible) {
        throw new Error(`Stock insuficiente para el producto: ${item.nombre}`);
      }
    }

    // 2. Calcular subtotal y total con ISV (15%)
    let subtotal = 0;
    for (let item of productos) {
      subtotal += item.precio * item.cantidad;
    }
    const ISV = subtotal * 0.15;
    const total = subtotal + ISV;

    // 3. Guardar en Base de Datos
    const query = "INSERT INTO pedidos (cliente_id, subtotal, isv, total) VALUES (?, ?, ?, ?)";
    this.db.query(query, [cliente.id, subtotal, ISV, total]);
    console.log("Pedido guardado en la base de datos.");

    // 4. Imprimir Ticket
    console.log("=== TICKET DE COMPRA ===");
    console.log(`Cliente: ${cliente.nombre}`);
    productos.forEach(item => {
      console.log(`${item.nombre} x${item.cantidad} - $${item.precio * item.cantidad}`);
    });
    console.log(`Subtotal: $${subtotal}`);
    console.log(`ISV (15%): $${ISV}`);
    console.log(`Total: $${total}`);
    console.log("=======================");

    // 5. Enviar mensaje de WhatsApp al cliente
    const mensaje = `Hola ${cliente.nombre}, tu pedido por un total de $${total} ha sido procesado con éxito.`;
    this.whatsapp.enviarMensaje(cliente.telefono, mensaje);
    console.log("Notificación enviada por WhatsApp.");

    return { exito: true, total };
  }
}