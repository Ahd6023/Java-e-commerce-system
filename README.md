import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

// Product interface
interface Product {
    String getName();
    double getPrice();
    int getQuantity();
    boolean isExpired();
    boolean requiresShipping();
    double getWeight();
}

// Concrete Product classes
class PerishableProduct implements Product {
    private String name;
    private double price;
    private int quantity;
    private double weight;
    private boolean expired;

    public PerishableProduct(String name, double price, int quantity, double weight, boolean expired) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
        this.weight = weight;
        this.expired = expired;
    }

    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public boolean isExpired() { return expired; }
    public boolean requiresShipping() { return true; }
    public double getWeight() { return weight; }
}

class NonPerishableProduct implements Product {
    private String name;
    private double price;
    private int quantity;
    private double weight;

    public NonPerishableProduct(String name, double price, int quantity, double weight) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
        this.weight = weight;
    }

    public String getName() { return name; }
    public double getPrice() { return price; }
    public int getQuantity() { return quantity; }
    public boolean isExpired() { return false; }
    public boolean requiresShipping() { return false; }
    public double getWeight() { return weight; }
}

// Shopping Cart class
class Cart {
    private Map<Product, Integer> items = new HashMap<>();

    public void add(Product product, int quantity) {
        if (quantity > product.getQuantity()) {
            throw new IllegalArgumentException("Requested quantity exceeds available stock.");
        }
        items.put(product, quantity);
    }

    public Map<Product, Integer> getItems() {
        return items;
    }

    public boolean isEmpty() {
        return items.isEmpty();
    }
}

// Customer class
class Customer {
    private String name;
    private double balance;

    public Customer(String name, double balance) {
        this.name = name;
        this.balance = balance;
    }

    public double getBalance() {
        return balance;
    }

    public void deductBalance(double amount) {
        balance -= amount;
    }
}

// Checkout class
class Checkout {
    private static final double SHIPPING_FEE = 30.0;

    public void processCheckout(Customer customer, Cart cart) {
        if (cart.isEmpty()) {
            throw new IllegalArgumentException("Cart is empty.");
        }

        double subtotal = 0;
        double totalWeight = 0;
        List<Product> shippableItems = new ArrayList<>();

        for (Map.Entry<Product, Integer> entry : cart.getItems().entrySet()) {
            Product product = entry.getKey();
            int quantity = entry.getValue();

            if (product.isExpired()) {
                throw new IllegalArgumentException("Product " + product.getName() + " is expired.");
            }

            if (product.getQuantity() < quantity) {
                throw new IllegalArgumentException("Product " + product.getName() + " is out of stock.");
            }

            subtotal += product.getPrice() * quantity;

            if (product.requiresShipping()) {
                totalWeight += product.getWeight() * quantity;
                shippableItems.add(product);
            }
        }

        double totalAmount = subtotal + SHIPPING_FEE;

        if (customer.getBalance() < totalAmount) {
            throw new IllegalArgumentException("Insufficient balance.");
        }

        customer.deductBalance(totalAmount);
        printReceipt(subtotal, totalWeight, totalAmount, shippableItems);
    }

    private void printReceipt(double subtotal, double totalWeight, double totalAmount, List<Product> shippableItems) {
        System.out.println("** Shipment notice **");
        for (Product product : shippableItems) {
            System.out.println("1x " + product.getName() + " " + product.getWeight() + "g");
        }
        System.out.println("Total package weight " + totalWeight + "kg");
        System.out.println("** Checkout receipt **");
        System.out.println("Subtotal: " + subtotal);
        System.out.println("Shipping: " + SHIPPING_FEE);
        System.out.println("Amount: " + totalAmount);
    }
}

// Main class to demonstrate functionality
public class ECommerceSystem {
    public static void main(String[] args) {
        // Create products
        Product cheese = new PerishableProduct("Cheese", 200, 10, 400, false);
        Product biscuits = new PerishableProduct("Biscuits", 150, 5, 700, false);
        Product tv = new NonPerishableProduct("TV", 1000, 2, 5000);
        Product mobileScratchCard = new NonPerishableProduct("Mobile Scratch Card", 50, 20, 0);

        // Create a customer
        Customer customer = new Customer("John Doe", 500);

        // Create a shopping cart
        Cart cart = new Cart();
        cart.add(cheese, 2);
        cart.add(biscuits, 1);
        cart.add(tv, 1);

        // Checkout
        Checkout checkout = new Checkout();
        checkout.processCheckout(customer, cart);
    }
}
