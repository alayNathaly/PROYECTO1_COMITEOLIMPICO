# PROYECTO1_COMITEOLIMPICO
Desarrolla una aplicación de consola que permita al Comité Olímpico Guatemalteco registrar y analizar el rendimiento de sus atletas de alto nivel.

<p align="center">
<img width="169" height="514" alt="Diagrama_Proyecto1" src="https://github.com/user-attachments/assets/c6b4a792-62a0-480e-95cd-e4919fde762a" />
</p>

```java
import java.io.*;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.*;

public class Olimpiadas {
    // clase atleta
    static class Athlete {
        private String nombre;
        private int edad;
        private double peso;    // kg
        private double altura;  // cm
        private String disciplina; // que deporte entrena
        private String departamento;  //en que departamento vive
        private List<TrainingSession> sesiones = new ArrayList<>(); //la lista de sesiones

        //constructor para datos atletas
        public Athlete(String nombre, int edad, double peso, double altura, String disciplina, String departamento) {
            this.nombre = nombre;
            this.edad = edad;
            this.peso = peso;
            this.altura = altura;
            this.disciplina = disciplina;
            this.departamento = departamento;
        }
        //metodos getters para acceder a los datod atleta
        public String getNombre() { return nombre; }
        public int getEdad() { return edad; }
        public double getPeso() { return peso; }
        public double getAltura() { return altura; }
        public String getDisciplina() { return disciplina; }
        public String getDepartamento() { return departamento; }
        public List<TrainingSession> getSesiones() { return sesiones; }

        public void addSesion(TrainingSession s) { sesiones.add(s); }

        // exportar a CSV
        public String toCsv() {
            return nombre + "," + edad + "," + peso + "," + altura + "," +
                    disciplina.replace(",", " ") + "," + departamento.replace(",", " ");
        }

        public static Athlete fromCsv(String line) {
            String[] p = line.split(",", -1);
            if (p.length < 6) return null;
            String n = p[0];
            int e = Integer.parseInt(p[1]);
            double pe = Double.parseDouble(p[2]);
            double al = Double.parseDouble(p[3]);
            String d = p[4];
            String dep = p[5];
            return new Athlete(n, e, pe, al, d, dep);
        }
    }

    // clase entrenamieto
    static class TrainingSession implements Comparable<TrainingSession> {
        private LocalDate fecha; //dia del entrenamiento
        private String tipo;    // p.ej. "pesas", "natación", etc.
        private double valor;   // si es con tiempo:en segundos; si es peso: kg
        private String unidad;  // "seg" o "kg"
        private String nota; //el atleta puede agregar una nota de su entrenamiento

        // formayo fecha
        private static final DateTimeFormatter FMT = DateTimeFormatter.ofPattern("dd-MM-yyyy");

        //constructor
        public TrainingSession(LocalDate fecha, String tipo, double valor, String unidad, String nota) {
            this.fecha = fecha;
            this.tipo = tipo;
            this.valor = valor;
            this.unidad = unidad;
            this.nota = nota;
        }
        //getters
        public LocalDate getFecha() { return fecha; }
        public String getTipo() { return tipo; }
        public double getValor() { return valor; }
        public String getUnidad() { return unidad; }
        public String getNota() { return nota; }

        //para ordenar las sesiones por sus fechas
        @Override
        public int compareTo(TrainingSession o) {
            return this.fecha.compareTo(o.fecha);
        }

        //exportar a CSV
        public String toCsv(String nombreAtleta) {
            return nombreAtleta + "," + fecha.format(FMT) + "," +
                    tipo.replace(",", " ") + "," + valor + "," +
                    unidad.replace(",", " ") + "," + nota.replace(",", " ");
        }
        //Convierte la feche en string
        public static String fechaToString(LocalDate f) {
            return f.format(FMT);
        }

        public static LocalDate parseFecha(String s) {
            return LocalDate.parse(s, FMT);
        }
    }

    // menu
    private static Scanner sc = new Scanner(System.in);
    private static Map<String, Athlete> atletas = new LinkedHashMap<>(); //se guardas todos los atletas registrados

    public static void main(String[] args) {
        int op;
        do {
            menu();
            op = leerEntero("Elige una opción: ");
            switch (op) {
                case 1: registrarAtleta(); break;
                case 2: registrarSesion(); break;      // << opción modificada
                case 3: mostrarHistorial(); break;
                case 4: mostrarEstadisticas(); break;
                case 5: guardarCSV(); break;
                case 6: cargarCSV(); break;
                case 7: exportarJSON(); break;
                case 0: System.out.println("Adios, regresa pronto!"); break;
                default: System.out.println("Opción inválida.");
            }
            System.out.println();
        } while (op != 0);
    }

    private static void menu() {
        System.out.println("=========== MENÚ ===========");
        System.out.println("1. Registrar al atleta");
        System.out.println("2. Registrar la sesión de entrenamiento");
        System.out.println("3. Ver historial del atleta");
        System.out.println("4. Ver estadísticas del atleta");
        System.out.println("5. Guardar a CSV");
        System.out.println("6. Cargar desde CSV");
        System.out.println("7. Exportar a JSON");
        System.out.println("0. Salir");
        System.out.println("============================");
    }

    // registrar atleta
    private static void registrarAtleta() {
        System.out.println("== Nuevo Atleta ==");
        String nombre = leerLinea("Nombre: ").trim();
        if (nombre.isEmpty()) return;
        if (atletas.containsKey(nombre)) {
            System.out.println("Upsi!! ya existe un atleta con ese nombre.");
            return;
        }
        int edad = leerEntero("Edad: ");
        double peso = leerDouble("Peso (kg): ");
        double altura = leerDouble("Altura (cm): ");
        String disciplina = leerLinea("Disciplina: ");
        String departamento = leerLinea("Departamento en Guatemala: ");

        Athlete a = new Athlete(nombre, edad, peso, altura, disciplina, departamento);
        atletas.put(nombre, a);
        System.out.println("Atleta registrado.");
    }

    // registrar sesion
    private static void registrarSesion() {
        if (atletas.isEmpty()) {
            System.out.println("No hay atletas :(.");
            return;
        }
        System.out.println("== Nueva Sesión ==");
        String nombre = leerLinea("Nombre del atleta: ");
        Athlete a = atletas.get(nombre);
        if (a == null) { System.out.println("ups!! no existe ese atleta"); return; }

        String fechaStr = leerLinea("Fecha (dd-MM-yyyy): ");
        LocalDate fecha;
        try { fecha = TrainingSession.parseFecha(fechaStr); }
        catch (Exception e) { System.out.println("Fecha inválida."); return; }

        String tipoEntrenamiento = leerLinea("Tipo de entrenamiento (ejemplo: pesas): ");
        // Elegir cómo se medirá el rendimiento
        String modo = leerLinea("¿Cómo mediras el rendimiento? tiempo opeso: ").trim().toLowerCase();

        double valor;
        String unidad;
        String notaExtra = "";

            //rendimiento por tiempo
        if (modo.startsWith("tiem")) {
            int min = leerEntero("Minutos: ");
            int seg = leerEntero("Segundos: ");
            if (min < 0) min = 0;
            if (seg < 0) seg = 0;
            valor = min * 60.0 + seg; // guardamos todo en segundos
            unidad = "seg";
            notaExtra = "Tiempo total = " + (int)valor + " seg";
        } else if (modo.startsWith("peso")) { //rendimiento por peso
            valor = leerDouble("Peso levantado (kg): ");
            unidad = "kg";
            notaExtra = "Peso en kg";
        } else {
            System.out.println("Opción inválida (usa 'tiempo' o 'peso').");
            return;
        }

        //notita que peude dejar el atleta
        String nota = leerLinea("Descripción/Prueba (opcional): ");
        if (!notaExtra.isEmpty()) {
            if (!nota.isEmpty()) nota += " | " + notaExtra;
            else nota = notaExtra;
        }

        TrainingSession s = new TrainingSession(fecha, tipoEntrenamiento, valor, unidad, nota);
        a.addSesion(s);
        System.out.println("Sesión registrada.");
    }

    // Historial
    private static void mostrarHistorial() {
        String nombre = leerLinea("Atleta: ");
        Athlete a = atletas.get(nombre);
        if (a == null) { System.out.println("No existe ese atleta."); return; }
        List<TrainingSession> lista = new ArrayList<>(a.getSesiones());
        if (lista.isEmpty()) { System.out.println("Sin sesiones."); return; }
        Collections.sort(lista); //se ordenas por fecha

        System.out.println("== Historial de " + nombre + " ==");
        for (TrainingSession s : lista) {
            System.out.println(TrainingSession.fechaToString(s.getFecha()) +
                    " | " + s.getTipo() + " | " + s.getValor() + " " + s.getUnidad() +
                    " | " + s.getNota());
        }
    }

    // calculos
    private static void mostrarEstadisticas() {
        String nombre = leerLinea("Atleta: ");
        Athlete a = atletas.get(nombre);
        if (a == null || a.getSesiones().isEmpty()) { System.out.println("No hay datos."); return; }

        // agrupamos por tipo de entrenamiento
        Map<String, List<TrainingSession>> porTipo = new LinkedHashMap<>();
        for (TrainingSession s : a.getSesiones())
            porTipo.computeIfAbsent(s.getTipo(), k -> new ArrayList<>()).add(s);

        for (String tipo : porTipo.keySet()) {
            List<TrainingSession> lista = porTipo.get(tipo);
            Collections.sort(lista);
            //calculo promedio
            double suma = 0;
            for (TrainingSession s : lista) suma += s.getValor();
            double promedio = suma / lista.size();

            // Mejor marca:
            // - "seg" => menor tiempo es mejor
            // - "kg"  => mayor peso es mejor
            TrainingSession mejor = lista.get(0);
            for (TrainingSession s : lista) {
                boolean tiempo = s.getUnidad().equalsIgnoreCase("seg");
                boolean mejorEsTiempo = mejor.getUnidad().equalsIgnoreCase("seg");
                if (tiempo && mejorEsTiempo) {
                    if (s.getValor() < mejor.getValor()) mejor = s;
                } else if (!tiempo && !mejorEsTiempo) {
                    if (s.getValor() > mejor.getValor()) mejor = s;
                } else {
                    // por si se llegan a mezclar unidades por error no se combinara
                }
            }

            //resultados
            System.out.println("== Tipo: " + tipo + " ==");
            System.out.printf("Promedio: %.2f %s%n", promedio, lista.get(0).getUnidad());
            System.out.println("Mejor: " + mejor.getValor() + " " + mejor.getUnidad() +
                    " (" + TrainingSession.fechaToString(mejor.getFecha()) + ")");
            System.out.println("Evolución:");
            for (TrainingSession s : lista)
                System.out.println("  " + TrainingSession.fechaToString(s.getFecha()) + " -> " + s.getValor() + " " + s.getUnidad());
        }
    }

    // Guardar CSV
    private static void guardarCSV() {
        try (PrintWriter pa = new PrintWriter(new FileWriter("atletas.csv"));
             PrintWriter ps = new PrintWriter(new FileWriter("sesiones.csv"))) {
            pa.println("nombre,edad,peso,altura,disciplina,departamento");
            ps.println("nombreAtleta,fecha,tipo,valor,unidad,nota");
            for (Athlete a : atletas.values()) { //aqui se escribien los datos
                pa.println(a.toCsv());
                for (TrainingSession s : a.getSesiones())
                    ps.println(s.toCsv(a.getNombre()));
            }
            System.out.println("Guardado en CSV.");
        } catch (IOException e) { System.out.println("Error: " + e.getMessage()); }
    }

    // Cargar CSV
    private static void cargarCSV() {
        Map<String, Athlete> cargados = new LinkedHashMap<>();
        try (BufferedReader br = new BufferedReader(new FileReader("atletas.csv"))) { //se cargan los atletas
            String line = br.readLine();
            while ((line = br.readLine()) != null) {
                Athlete a = Athlete.fromCsv(line);
                if (a != null) cargados.put(a.getNombre(), a);
            }
        } catch (Exception e) {}

        try (BufferedReader br = new BufferedReader(new FileReader("sesiones.csv"))) { //se cargan las sesiones
            String line = br.readLine();
            while ((line = br.readLine()) != null) {
                String[] p = line.split(",", -1);
                if (p.length < 6) continue;
                String nombre = p[0];
                Athlete a = cargados.get(nombre);
                if (a != null) {
                    LocalDate f = TrainingSession.parseFecha(p[1]); // también dd-MM-yyyy
                    TrainingSession s = new TrainingSession(f, p[2], Double.parseDouble(p[3]), p[4], p[5]);
                    a.addSesion(s);
                }
            }
        } catch (Exception e) {}

        atletas = cargados;
        System.out.println("Datos cargados desde CSV.");
    }

    // Exportar JSON
    private static void exportarJSON() {
        StringBuilder sb = new StringBuilder();
        sb.append("{ \"atletas\": [\n");
        int i = 0;
        for (Athlete a : atletas.values()) {
            sb.append("  { \"nombre\": \"").append(a.getNombre()).append("\", ");
            sb.append("\"edad\": ").append(a.getEdad()).append(", ");
            sb.append("\"peso\": ").append(a.getPeso()).append(", ");
            sb.append("\"altura\": ").append(a.getAltura()).append(", ");
            sb.append("\"disciplina\": \"").append(a.getDisciplina()).append("\", ");
            sb.append("\"departamento\": \"").append(a.getDepartamento()).append("\", ");
            sb.append("\"sesiones\": [");
            for (int j = 0; j < a.getSesiones().size(); j++) {
                TrainingSession s = a.getSesiones().get(j);
                sb.append("{\"fecha\": \"").append(TrainingSession.fechaToString(s.getFecha())).append("\", ");
                sb.append("\"tipo\": \"").append(s.getTipo()).append("\", ");
                sb.append("\"valor\": ").append(s.getValor()).append(", ");
                sb.append("\"unidad\": \"").append(s.getUnidad()).append("\", ");
                sb.append("\"nota\": \"").append(s.getNota()).append("\"}");
                if (j < a.getSesiones().size()-1) sb.append(",");
            }
            sb.append("]}");
            if (i < atletas.size()-1) sb.append(",");
            sb.append("\n");
            i++;
        }
        sb.append("]}");

        try (PrintWriter out = new PrintWriter(new FileWriter("datos.json"))) {
            out.print(sb.toString());
            System.out.println("Exportado a datos.json");
        } catch (IOException e) { System.out.println("Error: " + e.getMessage()); }
    }

    // Helpers
    private static String leerLinea(String msg) { //lee lineas
        System.out.print(msg);
        return sc.nextLine();
    }

    private static int leerEntero(String msg) { //validando errores
        while (true) {
            try {
                System.out.print(msg);
                return Integer.parseInt(sc.nextLine());
            } catch (Exception e) { System.out.println("Número inválido."); }
        }
    }

    private static double leerDouble(String msg) { //validadndo errores de los numeros
        while (true) {
            try {
                System.out.print(msg);
                return Double.parseDouble(sc.nextLine());
            } catch (Exception e) { System.out.println("Número inválido."); }
        }
    }
}
