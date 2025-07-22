# Student-Grade-Tracker
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.BorderLayout;
import java.awt.event.*;
import java.util.List;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;

public class StudentGradeTracker1 extends JFrame {

    private List<Student> students = new ArrayList<>();
    private DefaultTableModel tableModel;
    private JTable table;

    public StudentGradeTracker1() {
        setTitle("Student Grade Tracker");
        setSize(700, 500);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(new BorderLayout());

        // Table setup
        tableModel = new DefaultTableModel(new String[]{"Name", "ID", "Grade", "Status"}, 0);
        table = new JTable(tableModel);
        JScrollPane scrollPane = new JScrollPane(table);

        // Button panel
        JPanel buttonPanel = new JPanel();
        String[] buttons = {"Add Student", "Update Grade", "Delete Student", "Search", "Statistics", "Exit"};
        for (String text : buttons) {
            JButton btn = new JButton(text);
            btn.addActionListener(e -> handleAction(e.getActionCommand()));
            buttonPanel.add(btn);
        }

        add(new JLabel("Student Grade Tracker", SwingConstants.CENTER), BorderLayout.NORTH);
        add(scrollPane, BorderLayout.CENTER);
        add(buttonPanel, BorderLayout.SOUTH);

        setVisible(true);
    }

    private void handleAction(String command) {
        switch (command) {
            case "Add Student" -> addStudent();
            case "Update Grade" -> updateGrade();
            case "Delete Student" -> deleteStudent();
            case "Search" -> searchStudent();
            case "Statistics" -> showStatistics();
            case "Exit" -> System.exit(0);
        }
    }

    private void addStudent() {
        JTextField nameField = new JTextField();
        JTextField idField = new JTextField();
        JTextField gradeField = new JTextField();

        Object[] inputs = {
                "Name:", nameField,
                "ID:", idField,
                "Grade (0-100):", gradeField
        };

        int option = JOptionPane.showConfirmDialog(this, inputs, "Add Student", JOptionPane.OK_CANCEL_OPTION);
        if (option == JOptionPane.OK_OPTION) {
            try {
                String name = nameField.getText();
                String id = idField.getText();
                int grade = Integer.parseInt(gradeField.getText());

                if (grade < 0 || grade > 100) throw new NumberFormatException();

                Student student = new Student(name, id, grade);
                students.add(student);
                updateTable();
            } catch (NumberFormatException e) {
                JOptionPane.showMessageDialog(this, "Invalid grade. Please enter a number between 0-100.");
            }
        }
    }

    private void updateGrade() {
        String id = JOptionPane.showInputDialog(this, "Enter student ID to update:");
        Student student = findStudentById(id);
        if (student == null) {
            JOptionPane.showMessageDialog(this, "Student not found.");
            return;
        }

        String newGradeStr = JOptionPane.showInputDialog(this, "Enter new grade (0-100):");
        try {
            int newGrade = Integer.parseInt(newGradeStr);
            if (newGrade < 0 || newGrade > 100) throw new NumberFormatException();
            student.setGrade(newGrade);
            updateTable();
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(this, "Invalid grade input.");
        }
    }

    private void deleteStudent() {
        String id = JOptionPane.showInputDialog(this, "Enter student ID to delete:");
        Student student = findStudentById(id);
        if (student == null) {
            JOptionPane.showMessageDialog(this, "Student not found.");
            return;
        }

        students.remove(student);
        updateTable();
    }

    private void searchStudent() {
        String query = JOptionPane.showInputDialog(this, "Enter student name or ID:");
        if (query == null || query.trim().isEmpty()) return;

        List<Student> result = new ArrayList<>();
        for (Student s : students) {
            if (s.getName().toLowerCase().contains(query.toLowerCase()) ||
                    s.getId().toLowerCase().contains(query.toLowerCase())) {
                result.add(s);
            }
        }

        if (result.isEmpty()) {
            JOptionPane.showMessageDialog(this, "No matching students found.");
        } else {
            StringBuilder sb = new StringBuilder("Search Results:\n");
            for (Student s : result) {
                sb.append(s.getName()).append(" | ").append(s.getId())
                        .append(" | Grade: ").append(s.getGrade())
                        .append(" | Status: ").append(getGradeStatus(s.getGrade())).append("\n");
            }
            JOptionPane.showMessageDialog(this, sb.toString());
        }
    }

    private void showStatistics() {
        if (students.isEmpty()) {
            JOptionPane.showMessageDialog(this, "No students to calculate statistics.");
            return;
        }

        double average = students.stream().mapToInt(Student::getGrade).average().orElse(0.0);
        Student highest = Collections.max(students, Comparator.comparingInt(Student::getGrade));
        Student lowest = Collections.min(students, Comparator.comparingInt(Student::getGrade));

        String msg = String.format("""
                Grade Statistics:
                -------------------------
                Average: %.2f
                Highest: %s (%d)
                Lowest: %s (%d)
                -------------------------
                """, average,
                highest.getName(), highest.getGrade(),
                lowest.getName(), lowest.getGrade());

        JOptionPane.showMessageDialog(this, msg);
    }

    private Student findStudentById(String id) {
        for (Student s : students) {
            if (s.getId().equalsIgnoreCase(id)) return s;
        }
        return null;
    }

    private void updateTable() {
        tableModel.setRowCount(0);
        for (Student s : students) {
            tableModel.addRow(new Object[]{
                    s.getName(), s.getId(), s.getGrade(), getGradeStatus(s.getGrade())
            });
        }
    }

    private String getGradeStatus(int grade) {
        if (grade >= 90) return "A";
        if (grade >= 80) return "B";
        if (grade >= 70) return "C";
        if (grade >= 60) return "D";
        return "F";
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(StudentGradeTracker1::new);
    }

    // Inner Student class
    static class Student {
        private String name;
        private String id;
        private int grade;

        public Student(String name, String id, int grade) {
            this.name = name;
            this.id = id;
            this.grade = grade;
        }

        public String getName() { return name; }
        public String getId() { return id; }
        public int getGrade() { return grade; }
        public void setGrade(int grade) { this.grade = grade; }
    }
}
