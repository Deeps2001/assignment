import java.util.*;
class Product {
private int prodId;
private String prodName;
private double price;
private int quantity;
private double discount;
public Product(int prodId, String prodName, double price, int quantity, double discount) {
this.prodId = prodId;
this.prodName = prodName;
this.price = price;
this.quantity = quantity;
this.discount = discount;
}
}
class ProductNotFoundException extends Exception {
public ProductNotFoundException(String message) {
super(message);
}
}
interface ProductDAO {
void addProduct(Product product);
Product[] getAllProducts();
void deleteProduct(int prodId) throws ProductNotFoundException;
void updateProduct(Product product) throws ProductNotFoundException;
Product getProductById(int prodId) throws ProductNotFoundException;
Product[] getProductsByPriceAscending();
Product[] getProductsByPriceDescending();
Product[] getProductsByDiscountAscending();
Product[] getProductsByDiscountDescending();
}
class ProductDAOImpl implements ProductDAO {
private static final int MAX_PRODUCTS = 10;
private Product[] products = new Product[MAX_PRODUCTS];
private int productCount = 0;
@OverRide
public void addProduct(Product product) {
if (productCount < MAX_PRODUCTS) {
products[productCount++] = product;
System.out.println("Product added successfully.");
} else {
System.out.println("Cannot add more products. Database is full.");
}
}
@OverRide
public Product[] getAllProducts() {
return Arrays.copyOf(products, productCount);
}
@OverRide
public void deleteProduct(int prodId) throws ProductNotFoundException {
int foundIndex = -1;
for (int i = 0; i < productCount; i++) {
if (products[i].prodId == prodId) {
foundIndex = i;
break;
}
}
if (foundIndex != -1) {
for (int i = foundIndex; i < productCount - 1; i++) {
products[i] = products[i + 1];
}
productCount--;
System.out.println("Product deleted successfully.");
} else {
throw new ProductNotFoundException("Product with ID " + prodId + " not found.");
}
}
@OverRide
public void updateProduct(Product product) throws ProductNotFoundException {
int foundIndex = -1;
for (int i = 0; i < productCount; i++) {
if (products[i].prodId == product.prodId) {
foundIndex = i;
break;
}
}
if (foundIndex != -1) {
products[foundIndex] = product;
System.out.println("Product updated successfully.");
} else {
throw new ProductNotFoundException("Product with ID " + product.prodId + " not found.");
}
}
@OverRide
public Product getProductById(int prodId) throws ProductNotFoundException {
for (int i = 0; i < productCount; i++) {
if (products[i].prodId == prodId) {
return products[i];
}
}
throw new ProductNotFoundException("Product with ID " + prodId + " not found.");
}
@OverRide
public Product[] getProductsByPriceAscending() {
Product[] sortedProducts = Arrays.copyOf(products, productCount);
Arrays.sort(sortedProducts, Comparator.comparingDouble(product -> product.price));
return sortedProducts;
}
@OverRide
public Product[] getProductsByPriceDescending() {
Product[] sortedProducts = Arrays.copyOf(products, productCount);
Arrays.sort(sortedProducts, (product1, product2) -> Double.compare(product2.price, product1.price));
return sortedProducts;
}
@OverRide
public Product[] getProductsByDiscountAscending() {
Product[] sortedProducts = Arrays.copyOf(products, productCount);
Arrays.sort(sortedProducts, Comparator.comparingDouble(product -> product.discount));
return sortedProducts;
}
@OverRide
public Product[] getProductsByDiscountDescending() {
Product[] sortedProducts = Arrays.copyOf(products, productCount);
Arrays.sort(sortedProducts, (product1, product2) -> Double.compare(product2.discount, product1.discount));
return sortedProducts;
}
}
class ProductService {
private ProductDAO productDAO;

public ProductService(ProductDAO productDAO) {
    this.productDAO = productDAO;
}
public void addProduct(Product product) {
productDAO.addProduct(product);
}
public void displayAllProducts() {
Product[] products = productDAO.getAllProducts();
if (products.length == 0) {
System.out.println("No products available.");
} else {
System.out.println("All Products:");
for (Product product : products) {
System.out.println(product.prodId + " | " + product.prodName + " | " + product.price
+ " | " + product.quantity + " | " + product.discount);
}
}
}
public void deleteProduct(int prodId) {
try {
productDAO.deleteProduct(prodId);
System.out.println("Product deleted successfully.");
} catch (ProductNotFoundException e) {
System.out.println(e.getMessage());
}
}
public void updateProduct(Product product) {
try {
productDAO.updateProduct(product);
System.out.println("Product updated successfully.");
} catch (ProductNotFoundException e) {
System.out.println(e.getMessage());
}
}
public void searchProductById(int prodId) {
try {
Product product = productDAO.getProductById(prodId);
System.out.println("Product Found:");
System.out.println(product.prodId + " | " + product.prodName + " | " + product.price
+ " | " + product.quantity + " | " + product.discount);
} catch (ProductNotFoundException e) {
System.out.println(e.getMessage());
}
}
public void searchProductByName(String prodName) {
try {
Product[] products = productDAO.getProductsByDiscountDescending();
boolean productFound = false;
for (Product product : products) {
if (product.prodName.equalsIgnoreCase(prodName)) {
productFound = true;
System.out.println("Product Found:");
System.out.println(product.prodId + " | " + product.prodName + " | " + product.price
+ " | " + product.quantity + " | " + product.discount);
break;
}
}
if (!productFound) {
System.out.println("Product with name " + prodName + " not found.");
}
} catch (ProductNotFoundException e) {
System.out.println(e.getMessage());
}
}
public void displayProductsByPrice(boolean ascending) {
Product[] products;
if (ascending) {
products = productDAO.getProductsByPriceAscending();
} else {
products = productDAO.getProductsByPriceDescending();
}

    if (products.length == 0) {
        System.out.println("No products available.");
    } else {
        System.out.println("Products by " + (ascending ? "ascending" : "descending") + " price:");
        for (Product product : products) {
            System.out.println(product.prodId + " | " + product.prodName + " | " + product.price
                    + " | " + product.quantity + " | " + product.discount);
        }
    }
}
public void displayProductsByDiscount(boolean ascending) {
Product[] products;
if (ascending) {
products = productDAO.getProductsByDiscountAscending();
} else {
products = productDAO.getProductsByDiscountDescending();
}

    if (products.length == 0) {
        System.out.println("No products available.");
    } else {
        System.out.println("Products by " + (ascending ? "ascending" : "descending") + " discount:");
        for (Product product : products) {
            System.out.println(product.prodId + " | " + product.prodName + " | " + product.price
                    + " | " + product.quantity + " | " + product.discount);
        }
    }
}
}
public class Main {
public static void main(String[] args) {
Random rand = new Random();

    ProductDAO productDAO = new ProductDAOImpl();
    ProductService productService = new ProductService(productDAO);
    for (int i = 0; i < 10; i++) {
        int prodId = i + 1;
        String prodName = "Product " + prodId;
        double price = rand.nextDouble() * 100;
        int quantity = rand.nextInt(50) + 1;
        double discount = rand.nextDouble() * 0.5;Product product = new Product(prodId, prodName, price, quantity, discount);
        productService.addProduct(product);
    }
    productService.displayAllProducts();
    productService.deleteProduct(3);
    Product updatedProduct = new Product(5, "Updated Product", 75.0, 30, 0.2);
    productService.updateProduct(updatedProduct);
    productService.searchProductById(2);
    productService.searchProductByName("Updated Product");
    productService.displayProductsByPrice(true);
    productService.displayProductsByDiscount(false);
}
}
