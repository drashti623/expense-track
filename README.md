package in.ac.adit.pwj.miniproject.expenses;

import java.io.*;
import java.text.SimpleDateFormat;
import java.util.*;

public class ExpenseTrack {
    
    // Expense base class
    abstract class Expense {
        protected String description;
        protected double amount;
        protected String date;

        public Expense(String description, double amount, String date) {
            this.description = description;
            this.amount = amount;
            this.date = date;
        }

        public abstract String getCategory();

        @Override
        public String toString() {
            return String.format("%s: ₹%.2f - %s on %s", getCategory(), amount, description, date);
        }
    }

    // FoodExpense class inheriting from Expense
    class FoodExpense extends Expense {
        public FoodExpense(String description, double amount, String date) {
            super(description, amount, date);
        }

        @Override
        public String getCategory() {
            return "Food";
        }
    }

    // TravelExpense class inheriting from Expense
    class TravelExpense extends Expense {
        public TravelExpense(String description, double amount, String date) {
            super(description, amount, date);
        }

        @Override
        public String getCategory() {
            return "Travel";
        }
    }

    // Collection of expenses
    private List<Expense> expenses = new ArrayList<>();
    private Map<String, Double> categoryBudgets = new HashMap<>();

    // Constructor to initialize category budgets
    public ExpenseTrack() {
        categoryBudgets.put("Food", 5000.0);  // Example budget for Food
        categoryBudgets.put("Travel", 3000.0);  // Example budget for Travel
    }

    // Method to add an expense
    public void addExpense(Expense expense) throws Exception {
        double currentTotal = expenses.stream()
                .filter(e -> e.getCategory().equals(expense.getCategory()))
                .mapToDouble(e -> e.amount)
                .sum();

        double budget = categoryBudgets.getOrDefault(expense.getCategory(), Double.MAX_VALUE);

        if (currentTotal + expense.amount > budget) {
            throw new Exception("Budget overrun for category: " + expense.getCategory());
        }

        expenses.add(expense);
    }

    // Inner class for generating reports
    class ReportGenerator {
        public void generateReport() {
            String reportData = generateReportData();

            // Write the reportData to a .txt file
            try (BufferedWriter writer = new BufferedWriter(new FileWriter("ExpenseReport.txt"))) {
                writer.write(reportData);
                System.out.println("Report generated and saved to ExpenseReport.txt");
            } catch (IOException e) {
                System.out.println("Error writing to file: " + e.getMessage());
            }
        }

        // Method to generate the report data as a String
        private String generateReportData() {
            StringBuilder report = new StringBuilder();
            report.append("                    Expense Report\n");
            report.append("===================================================\n");
            report.append(" Sr. No. | Date       | Expense Type     |   Amount\n");
            report.append("===================================================\n");

            double totalExpense = 0.0;

            // Add expense details with Sr. No.
            int srNo = 1;  // Start serial number from 1
            for (Expense expense : expenses) {
                totalExpense += expense.amount;
                report.append(String.format("%-8d | %s | %-16s |    %.2f\n", srNo++, expense.date, expense.getCategory(), expense.amount));
            }

            report.append("====================================================\n");
            report.append(String.format("               Total Expenses:  %.2f\n", totalExpense));
            return report.toString();
        }
    }

    // Method to log user expenses
    public void logUserExpenses() {
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.print("Enter expense description (or type 'exit' to finish): ");
            String description = scanner.nextLine();
            if (description.equalsIgnoreCase("exit")) {
                break;
            }

            System.out.print("Enter expense amount: ");
            double amount;
            while (true) {
                try {
                    amount = Double.parseDouble(scanner.nextLine());
                    if (amount <= 0) {
                        throw new NumberFormatException();
                    }
                    break;
                } catch (NumberFormatException e) {
                    System.out.print("Please enter a valid positive number for the amount: ");
                }
            }

            System.out.print("Enter the date (DD-MM-YYYY): ");
            String date;
            while (true) {
                date = scanner.nextLine();
                // Validate date format
                if (isValidDate(date)) {
                    break;
                } else {
                    System.out.print("Please enter a valid date in the format DD-MM-YYYY: ");
                }
            }

            System.out.print("Choose a category (1: Food, 2: Travel, 3: Custom): ");
            String choice = scanner.nextLine();
            String category;

            if (choice.equals("1")) {
                category = "Food";
            } else if (choice.equals("2")) {
                category = "Travel";
            } else {
                System.out.print("Enter custom category name: ");
                category = scanner.nextLine();
            }

            try {
                Expense expense;
                if (category.equals("Food")) {
                    expense = new FoodExpense(description, amount, date);
                } else if (category.equals("Travel")) {
                    expense = new TravelExpense(description, amount, date);
                } else {
                    expense = new Expense(description, amount, date) {
                        @Override
                        public String getCategory() {
                            return category;
                        }
                    };
                }
                addExpense(expense);
                System.out.println("Expense added successfully.");
            } catch (Exception e) {
                System.out.println(e.getMessage());
            }
        }
        scanner.close();
    }

    // Method to validate date format
    private boolean isValidDate(String dateStr) {
        SimpleDateFormat sdf = new SimpleDateFormat("dd-MM-yyyy");
        sdf.setLenient(false);
        try {
            sdf.parse(dateStr);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // Main method to simulate concurrent expense logging and report generation
    public static void main(String[] args) {
        ExpenseTrack tracker = new ExpenseTrack();
        
        // Log user expenses
        tracker.logUserExpenses();

        // Generate the report
        tracker.new ReportGenerator().generateReport();
    }
}
 
