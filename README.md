Login.java
Este formulario permite al usuario iniciar sesión utilizando credenciales almacenadas en la base de datos.
Es el punto de entrada al sistema y controla el acceso a las demás funcionalidades.

Funcionamiento:
El usuario ingresa su nombre de usuario y contraseña.
El sistema se conecta a la base de datos usando la clase ConexionDB.

Se verifica:
Si el usuario existe y si está activo
El número de intentos fallidos

Si la contraseña es incorrecta:
Se incrementa el contador de intentos
El usuario se bloquea al tercer intento

Si las credenciales son correctas:
Se reinician los intentos
Se abre el formulario principal BancoForm

Código Login.java:
import javax.swing.*;
import java.sql.*;

public class Login extends JFrame {
    private JPanel Login_pa;
    private JTextField txtUsuario;
    private JTextField txtContraseña;
    private JButton btnIngresar;

    public Login() {
        setTitle("Inicio de Sesion");
        setContentPane(Login_pa);
        setSize(400, 250);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setVisible(true);

        // Botón ingresar
        btnIngresar.addActionListener(e -> init());
    }

    // Método de validación del login
    public void init() {
        String usuario = txtUsuario.getText();
        String contra = txtContraseña.getText();

        try {
            // Conexión a la base de datos
            Connection con = ConexionDB.getConexion();

            // Consulta para obtener datos del usuario
            String sql = """
                SELECT password, activo, intentos
                FROM usuarios_cliente
                WHERE username = ?
            """;

            PreparedStatement ps = con.prepareStatement(sql);
            ps.setString(1, usuario);
            ResultSet rs = ps.executeQuery();

            // Si el usuario no existe
            if (!rs.next()) {
                JOptionPane.showMessageDialog(this, "Usuario no registrado");
                return;
            }

            boolean activo = rs.getBoolean("activo");
            int intentos = rs.getInt("intentos");
            String passBD = rs.getString("password");

            if (!activo) {
                JOptionPane.showMessageDialog(this, "Usuario bloqueado");
                return;
            }
            
            // Si la contraseña es incorrecta
            if (!passBD.equals(contra)) {
                intentos++;

                String update = """
                    UPDATE usuarios_cliente
                    SET intentos = ?, activo = ?
                    WHERE username = ?
                """;

                PreparedStatement ups = con.prepareStatement(update);
                ups.setInt(1, intentos);
                ups.setBoolean(2, intentos < 3);
                ups.setString(3, usuario);
                ups.executeUpdate();

                JOptionPane.showMessageDialog(
                        this,
                        "Contraseña incorrecta. Intento " + intentos + " de 3"
                );
                return;
            }

            // Si el login es correcto, se reinician los intentos
            String reset = """
                UPDATE usuarios_cliente
                SET intentos = 0
                WHERE username = ?
            """;

            PreparedStatement resetPs = con.prepareStatement(reset);
            resetPs.setString(1, usuario);
            resetPs.executeUpdate();

            JOptionPane.showMessageDialog(this, "Registro exitoso.");
            dispose();
            new BancoForm(); // saldo = 1000

        } catch (SQLException e) {
            JOptionPane.showMessageDialog(this, "Error de conexión con la base de datos");
            e.printStackTrace();
        }
    }
}

<img width="653" height="408" alt="image" src="https://github.com/user-attachments/assets/72505a52-f1e8-4710-bd7b-5b2e25306a04" />
<img width="836" height="305" alt="image" src="https://github.com/user-attachments/assets/159b279e-fdd1-4c54-9de2-fdb15ca3b685" />
<img width="803" height="301" alt="image" src="https://github.com/user-attachments/assets/dfe0c487-919b-4d42-9ecb-207e7b8eef86" />

BancoForm.java
Es el formulario principal del sistema, donde el usuario puede ver su saldo y acceder a las operaciones bancarias.

Funcionamiento:
Muestra el saldo actual almacenado en la variable estática saldo.

Contiene botones para:
Depositar
Retirar
Transferir
Salir del sistema
Cada botón abre un formulario diferente.
El saldo se maneja mediante una variable estática, lo que permite que todos los formularios compartan y actualicen el mismo valor.

Código BancoForm.java:
import javax.swing.*;

public class BancoForm extends JFrame {

    // Componentes del formulario
    private JPanel BancoPanel;
    private JButton depositarButton;
    private JButton retirarButton;
    private JButton salirButton;
    private JButton transferenciaButton;
    private JTextField textUsuario;
    private JTextField TextFieldSaldoI;
    private JLabel JLabelBienvenido;
    private JLabel JLabelSaldo;

    // Variable estática para compartir el saldo entre formularios
    public static double saldo = 1000;

    public BancoForm() {
        // Título de la ventana
        setTitle("Banco");

        // Panel principal del formulario
        setContentPane(BancoPanel);

        // Tamaño de la ventana
        setSize(500, 350);

        // Cierra el programa al cerrar la ventana
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        // Hace visible el formulario
        setVisible(true);

        // Los campos no pueden ser editados por el usuario
        textUsuario.setEditable(false);
        TextFieldSaldoI.setEditable(false);

        // Se muestran los datos iniciales
        textUsuario.setText("      CLIENTE 1");
        TextFieldSaldoI.setText(String.valueOf(saldo));

        // Acciones de los botones
        depositarButton.addActionListener(e -> abrirDeposito());
        retirarButton.addActionListener(e -> abrirRetiro());
        transferenciaButton.addActionListener(e -> abrirTransferencia());
        salirButton.addActionListener(e -> System.exit(0));
    }

    // Abre el formulario de depósito
    private void abrirDeposito() {
        dispose(); // Cierra la ventana actual
        new DepositarForm(); // Abre el formulario de depósito
    }

    // Abre el formulario de retiro
    private void abrirRetiro() {
        dispose();
        new RetirarForm();
    }

    // Abre el formulario de transferencia
    private void abrirTransferencia() {
        dispose();
        new TransferenciaForm();
    }

    // Actualiza el campo del saldo
    public static void actualizarSaldo(JTextField field) {
        field.setText(String.valueOf(saldo));
    }
}

<img width="604" height="412" alt="image" src="https://github.com/user-attachments/assets/19e31e67-d233-40db-8166-414bcb0235a8" />

DepositarForm.java
Permite al usuario depositar dinero en su cuenta.

Funcionamiento
El usuario ingresa un monto.
Se valida que el monto sea mayor a cero.
El monto se suma al saldo actual.
Se muestra un mensaje de confirmación.
Se regresa al formulario principal.

Modifica directamente la variable BancoForm.saldo.

Código DepositarForm.java:
import javax.swing.*;

public class DepositarForm extends JFrame {

    private JTextField MontoTextField;
    private JButton confirmarButton;
    private JButton cancelarButton;
    private JLabel JLabelIMonto;
    private JPanel DepositarPanel;

    public DepositarForm() {
        setTitle("Depositar");
        setContentPane(DepositarPanel);
        setSize(300, 200);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setVisible(true);

        // Acción del botón confirmar
        confirmarButton.addActionListener(e -> depositar());

        // Acción del botón cancelar
        cancelarButton.addActionListener(e -> salir());
    }

    // Método para realizar el depósito
    private void depositar() {
        try {
            double monto = Double.parseDouble(MontoTextField.getText());

            // Validación del monto
            if (monto <= 0) {
                JOptionPane.showMessageDialog(this, "Monto invalido");
                return;
            }

            // Se suma el monto al saldo
            BancoForm.saldo += monto;

            JOptionPane.showMessageDialog(this, "Deposito exitoso");

            dispose();
            new BancoForm();

        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "ERROR >:|");
        }
    }

    // Regresa al menú principal
    private void salir() {
        dispose();
        new BancoForm();
    }
}

import javax.swing.*;

public class DepositarForm extends JFrame {

    private JTextField MontoTextField;
    private JButton confirmarButton;
    private JButton cancelarButton;
    private JLabel JLabelIMonto;
    private JPanel DepositarPanel;

    public DepositarForm() {
        setTitle("Depositar");
        setContentPane(DepositarPanel);
        setSize(300, 200);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setVisible(true);

        // Acción del botón confirmar
        confirmarButton.addActionListener(e -> depositar());

        // Acción del botón cancelar
        cancelarButton.addActionListener(e -> salir());
    }

    // Método para realizar el depósito
    private void depositar() {
        try {
            double monto = Double.parseDouble(MontoTextField.getText());

            // Validación del monto
            if (monto <= 0) {
                JOptionPane.showMessageDialog(this, "Monto invalido");
                return;
            }

            // Se suma el monto al saldo
            BancoForm.saldo += monto;

            JOptionPane.showMessageDialog(this, "Deposito exitoso");

            dispose();
            new BancoForm();

        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "ERROR >:|");
        }
    }

    // Regresa al menú principal
    private void salir() {
        dispose();
        new BancoForm();
    }
}

<img width="630" height="217" alt="image" src="https://github.com/user-attachments/assets/9919f083-583c-4c1f-99de-235cb28d1508" />
<img width="558" height="347" alt="image" src="https://github.com/user-attachments/assets/b402baf5-0d2f-482a-b7bd-c1b606bbf9be" />

RetirarForm.java
Permite al usuario retirar dinero de su cuenta.

Funcionamiento
Se valida que el monto sea mayor a cero.
Se verifica que el saldo sea suficiente.
Se descuenta el monto del saldo.
Se muestra un mensaje de retiro exitoso.
Retorna al menú principal.
Utiliza y modifica el saldo compartido del sistema.

Código RetirarForm.java:
import javax.swing.*;

public class RetirarForm extends JFrame {

    private JTextField MontoTextField;
    private JButton confirmarButton;
    private JButton cancelarButton;
    private JLabel JLabelPMonto;
    private JPanel RetirarPanel;

    public RetirarForm() {
        setTitle("Retirar");
        setContentPane(RetirarPanel);
        setSize(300, 200);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setVisible(true);

        confirmarButton.addActionListener(e -> retirar());
        cancelarButton.addActionListener(e -> salir());
    }

    // Método para retirar dinero
    private void retirar() {
        try {
            double monto = Double.parseDouble(MontoTextField.getText());

            if (monto <= 0) {
                JOptionPane.showMessageDialog(this, "Monto invalido");
                return;
            }

            if (monto > BancoForm.saldo) {
                JOptionPane.showMessageDialog(this, "Saldo insuficiente");
                return;
            }

            BancoForm.saldo -= monto;
            JOptionPane.showMessageDialog(this, "Retiro exitoso");

            dispose();
            new BancoForm();

        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "ERROR >:|");
        }
    }

    private void salir() {
        dispose();
        new BancoForm();
    }
}

<img width="636" height="230" alt="image" src="https://github.com/user-attachments/assets/ea430c37-b597-4918-a4b6-02a511fcd9b5" />
<img width="639" height="398" alt="image" src="https://github.com/user-attachments/assets/9e8ab2bf-a3fc-48cd-b17d-97a78cf2f336" />

TransferenciaForm.java
Permite realizar una transferencia a otro destinatario.

Funcionamiento
Se valida que el destinatario no esté vacío.
Se valida el monto ingresado.
Se verifica el saldo disponible.
Se descuenta el monto del saldo.
Se muestra un mensaje confirmando la transferencia.
Comparte la lógica del saldo con los demás formularios.

Código TransferenciaForm.java:
import javax.swing.*;

public class TransferenciaForm extends JFrame {

    private JTextField textIDatos;
    private JTextField textMDatos;
    private JTextField MontoTextField;
    private JButton confirmarButton;
    private JButton cancelarButton;
    private JButton validarButton;
    private JLabel JLabelMonto;
    private JLabel JLabelMDatos;
    private JLabel JLabelIDatos;
    private JPanel TransferenciaPanel;

    public TransferenciaForm() {
        setTitle("Transferencia");
        setContentPane(TransferenciaPanel);
        setSize(350, 260);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setVisible(true);

        validarButton.addActionListener(e -> validar());
        confirmarButton.addActionListener(e -> transferir());
        cancelarButton.addActionListener(e -> salir());
    }

    // Valida que el destinatario esté ingresado
    private void validar() {
        if (textIDatos.getText().isEmpty()) {
            JOptionPane.showMessageDialog(this, "Ingrese destinatario");
        } else {
            JOptionPane.showMessageDialog(this, "Destinatario valido");
        }
    }

    // Realiza la transferencia
    private void transferir() {
        try {
            String destinatario = textIDatos.getText();
            double monto = Double.parseDouble(MontoTextField.getText());

            if (destinatario.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Ingrese destinatario");
                return;
            }

            if (monto <= 0) {
                JOptionPane.showMessageDialog(this, "Monto invalido");
                return;
            }

            if (monto > BancoForm.saldo) {
                JOptionPane.showMessageDialog(this, "Saldo insuficiente");
                return;
            }

            BancoForm.saldo -= monto;

            JOptionPane.showMessageDialog(this,
                    "Transferencia realizada a " + destinatario + " por $" + monto);

            dispose();
            new BancoForm();

        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "ERROR >:|");
        }
    }

    private void salir() {
        dispose();
        new BancoForm();
    }
}

<img width="767" height="327" alt="image" src="https://github.com/user-attachments/assets/7fe4e74d-c1c3-4d8a-a06c-be8e8ac032df" />
<img width="644" height="417" alt="image" src="https://github.com/user-attachments/assets/74631a34-0de3-48e2-9be8-e0e3cc77be17" />

Main.java
Es el punto de inicio del programa.

Funcionamiento
Verifica la conexión con la base de datos.
Muestra mensajes en consola según el estado de la conexión.
Abre el formulario de inicio de sesión.

Código Main.java:
import java.sql.Connection;

public class Main {

    public static void main(String[] args) {

        // Verificación de conexión a la base de datos
        try {
            Connection con = ConexionDB.getConexion();
            System.out.println("Conexión exitosa a PostgreSQL");
        } catch (Exception e) {
            System.out.println("Error de conexión");
            e.printStackTrace();
        }

        // Se abre el formulario de login
        new Login();
    }
}

ConexionDB.java
Establece la conexión con la base de datos PostgreSQL.

Funcionamiento
Usa DriverManager para conectarse a la base de datos.
Implementa un patrón de conexión reutilizable.
Garantiza una sola conexión activa.
Es utilizada principalmente por el formulario Login.

Código ConexionDB.java:
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConexionDB {

    // Datos de conexión a PostgreSQL
    private static final String URL =
            "jdbc:postgresql://localhost:5432/almacen";
    private static final String USER = "postgres";
    private static final String PASSWORD = "12345";

    // Variable de conexión
    private static Connection conexion;

    // Método que devuelve la conexión
    public static Connection getConexion() throws SQLException {

        // Si no hay conexión o está cerrada, se crea una nueva
        if (conexion == null || conexion.isClosed()) {
            conexion = DriverManager.getConnection(URL, USER, PASSWORD);
        }

        return conexion;
    }
}

Creación de la base de datos con su respectivo usuario
La base de datos incluye una función que permite desbloquear al usuario cuando este ha sido bloqueado debido a intentos incorrectos de autenticación.

<img width="534" height="403" alt="image" src="https://github.com/user-attachments/assets/2d30c5b3-e0a1-44e1-8d88-52ecca29b46f" />
