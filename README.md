using System;
using System.Collections.Generic;

namespace GestorVentasUnidad1
{
    class Program
    {
        static void Main(string[] args)
        {
            List<string> nombresProductos = new List<string>();
            List<decimal> preciosProductos = new List<decimal>();
            List<int> stocksProductos = new List<int>();
            List<int> ventasPorProducto = new List<int>();

            int totalVentasSesion = 0;
            decimal totalDineroCaja = 0m;
            bool ejecutando = true;

            do
            {
                Console.Clear();
                Console.WriteLine("====================================================");
                Console.WriteLine("SISTEMA GESTOR DE VENTAS E INVENTARIO");
                Console.WriteLine("1. registre un nuevo producto al inventario");
                Console.WriteLine("2. consultar inventario");
                Console.WriteLine("3. registrar una venta");
                Console.WriteLine("4. reporte de caja y estadisticas");
                Console.WriteLine("5. salir");
                Console.WriteLine("====================================================");

                int opcion = LeerEntero("selecciona una opcion (1-5): ", 1, 5);

                switch (opcion)
                {
                    case 1:
                        RegistrarProducto(nombresProductos, preciosProductos, stocksProductos, ventasPorProducto);
                        break;

                    case 2:
                        ConsultarInventario(nombresProductos, preciosProductos, stocksProductos);
                        break;

                    case 3:
                        RegistrarVenta(nombresProductos, preciosProductos, stocksProductos, ventasPorProducto, ref totalVentasSesion, ref totalDineroCaja);
                        break;

                    case 4:
                        VerReporteCaja(nombresProductos, ventasPorProducto, totalVentasSesion, totalDineroCaja);
                        break;

                    case 5:
                        ejecutando = false;
                        Console.WriteLine("\n¡Gracias por usar el sistema!...");
                        Console.ReadKey();
                        break;
                }
            } while (ejecutando);
        }
        static int LeerEntero(string mensaje, int min, int max)
        {
            int numero;
            bool esValido = false;

            do
            {
                Console.Write(mensaje);

                if (int.TryParse(Console.ReadLine(), out numero) &&
                    numero >= min && numero <= max)
                    esValido = true;
                else
                    Console.WriteLine($"[ERROR] ingresa un numero entre {min} y {max}.");

            } while (!esValido);

            return numero;
        }
        static decimal LeerDecimal(string mensaje, decimal min)
        {
            decimal numero;
            bool esValido = false;

            do
            {
                Console.Write(mensaje);

                if (decimal.TryParse(Console.ReadLine(), out numero) && numero >= min)
                    esValido = true;
                else
                    Console.WriteLine($"[ERROR] ingresa un valor mayor o igual a {min:C2}.");

            } while (!esValido);

            return numero;
        }
        static decimal CalcularFactura(decimal precio, int cantidad, bool tieneDescuento,out decimal montoIva, out decimal montoDescuento)
        {
            decimal subtotal = precio * cantidad;

            montoDescuento = tieneDescuento ? subtotal * 0.10m : 0m;
            decimal subtotalConDescuento = subtotal - montoDescuento;

            montoIva = subtotalConDescuento * 0.19m;

            return subtotalConDescuento + montoIva;
        }
        static void ImprimirEncabezado(string titulo)
        {
            Console.WriteLine("====================================================");
            Console.WriteLine($"   {titulo}");
            Console.WriteLine("====================================================");
        }
        static void RegistrarProducto(List<string> nombres, List<decimal> precios,
            List<int> stocks, List<int> ventas)
        {
            Console.Clear();
            ImprimirEncabezado("REGISTRAR NUEVO PRODUCTO");

            string nombre;
            bool nombreValido = false;

            do
            {
                Console.Write("nombre del producto: ");
                nombre = Console.ReadLine()?.Trim() ?? "";

                if (string.IsNullOrWhiteSpace(nombre))
                {
                    Console.WriteLine("[ERROR] el nombre no puede estar vacio.");
                    continue;
                }

                bool existe = false;

                foreach (string item in nombres)
                {
                    if (item.Equals(nombre, StringComparison.OrdinalIgnoreCase))
                    {
                        existe = true;
                        break;
                    }
                }

                if (existe)
                    Console.WriteLine("[ERROR] ya existe un producto con este nombre.");
                else
                    nombreValido = true;

            } while (!nombreValido);

            decimal precio = LeerDecimal("precio unitario ($): ", 0.01m);
            int stock = LeerEntero("stock inicial: ", 0, int.MaxValue);

            nombres.Add(nombre);
            precios.Add(precio);
            stocks.Add(stock);
            ventas.Add(0);

            Console.WriteLine("\n[OK] producto registrado correctamente.");
            Pausar();
        }

        static void ConsultarInventario(List<string> nombres, List<decimal> precios,
            List<int> stocks)
        {
            Console.Clear();
            ImprimirEncabezado("INVENTARIO COMPLETO");

            if (nombres.Count == 0)
                Console.WriteLine("no hay productos registrados en el inventario.");
            else
            {
                for (int i = 0; i < nombres.Count; i++)
                {
                    string alerta = stocks[i] < 5 ? " [ALERTA: BAJO STOCK]" : "";

                    Console.WriteLine($"{i + 1}. {nombres[i],-25} | Precio: {precios[i],12:C2} | Stock: {stocks[i],4}{alerta}");
                }
            }
            Pausar();
        }
        static void RegistrarVenta(List<string> nombres, List<decimal> precios,
            List<int> stocks, List<int> ventas, ref int totalVentas, ref decimal totalCaja)
        {
            Console.Clear();
            ImprimirEncabezado("REGISTRAR VENTA");

            if (nombres.Count == 0)
            {
                Console.WriteLine("no hay productos en inventario para vender.");
                Pausar();
                return;
            }

            for (int i = 0; i < nombres.Count; i++)
            {
                string alerta = stocks[i] < 5 ? " [ALERTA: BAJO STOCK]" : "";
                Console.WriteLine($"{i + 1}. {nombres[i],-25} | Precio: {precios[i],12:C2} | Stock: {stocks[i],4}{alerta}");
            }

            Console.WriteLine();

            int seleccion = LeerEntero(
                $"selecciona el numero del producto a vender (1-{nombres.Count}): ",1, nombres.Count);

            int indice = seleccion - 1;

            if (stocks[indice] == 0)
            {
                Console.WriteLine($"\n[ERROR] el producto '{nombres[indice]}' se encuentra agotado.");
                Pausar();
                return;
            }

            int cantidad;

            do
            {
                cantidad = LeerEntero("ingresa la cantidad a comprar: ", 1, int.MaxValue);

                if (cantidad > stocks[indice])
                    Console.WriteLine($"[ERROR] stock insuficiente. quedan {stocks[indice]} unidades en inventario.");

            } while (cantidad > stocks[indice]);

            bool tieneDescuento = false;
            bool respuestaValida = false;

            do
            {
                Console.Write("¿aplica descuento de cliente frecuente (10%)? (S/N): ");
                string respuesta = Console.ReadLine()?.Trim().ToUpper() ?? "";

                if (respuesta == "S" || respuesta == "N")
                {
                    tieneDescuento = respuesta == "S";
                    respuestaValida = true;
                }
                else
                    Console.WriteLine("[ERROR] entrada invalida. ingresa 'S' o 'N'.");

            } while (!respuestaValida);

            decimal subtotal = precios[indice] * cantidad;
            decimal montoIva;
            decimal montoDescuento;

            decimal totalPagar = CalcularFactura(precios[indice], cantidad, tieneDescuento,out montoIva, out montoDescuento);

            stocks[indice] -= cantidad;
            ventas[indice] += cantidad;
            totalVentas++;
            totalCaja += totalPagar;

            Console.WriteLine();
            ImprimirEncabezado("TICKET DE VENTA");
            Console.WriteLine($" producto:             {nombres[indice]} (x{cantidad})");
            Console.WriteLine($" subtotal:             {subtotal,12:C2}");
            Console.WriteLine($" descuento (10%):     -{montoDescuento,12:C2}");
            Console.WriteLine($" IVA (19%):           +{montoIva,12:C2}");
            Console.WriteLine(" ---------------------------------------------------");
            Console.WriteLine($" TOTAL A PAGAR:        {totalPagar,12:C2}");
            Console.WriteLine("====================================================");
            Console.WriteLine($"[OK] venta realizada con exito. stock actualizado: {stocks[indice]} unidades.");
            Pausar();
        }
        static void VerReporteCaja(List<string> nombres, List<int> ventas,int totalVentas, decimal totalCaja)
        {
            Console.Clear();
            ImprimirEncabezado("REPORTE DE CAJA Y ESTADÍSTICAS DIARIAS");

            Console.WriteLine($"Total de ventas realizadas:      {totalVentas}");
            Console.WriteLine($"Total acumulado en caja:         {totalCaja:C2}");

            decimal promedio = totalVentas > 0 ? totalCaja / totalVentas : 0m;
            Console.WriteLine($"Promedio por venta:              {promedio:C2}");

            if (totalVentas > 0)
            {
                int maxVendido = -1;
                string productoMasVendido = "ninguno";

                for (int i = 0; i < ventas.Count; i++)
                {
                    if (ventas[i] > maxVendido)
                    {
                        maxVendido = ventas[i];
                        productoMasVendido = nombres[i];
                    }
                }

                Console.WriteLine($"producto mas vendido:            {productoMasVendido} ({maxVendido} unidades)");
            }
            else
                Console.WriteLine("producto mas vendido:            N/A (sin ventas registradas)");

            Pausar();
        }
        static void Pausar()
        {
            Console.WriteLine("\npresione una tecla para continuar...");
            Console.ReadKey();
        }
    }
}
