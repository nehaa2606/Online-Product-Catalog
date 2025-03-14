# Online-Product-Catalog
Create an e-commerce product listing page. Allow users to search and filter products stored in a database.
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(100),
    price DECIMAL(10,2),
    stock INT,
    image_url VARCHAR(255)
);
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DatabaseConnection {
    private static final String URL = "jdbc:mysql://localhost:3306/ecommerce";
    private static final String USER = "root";
    private static final String PASSWORD = "";

    public static Connection getConnection() throws SQLException {
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
}
public class Product {
    private int id;
    private String name;
    private String description;
    private String category;
    private double price;
    private int stock;
    private String imageUrl;

    public Product(int id, String name, String description, String category, double price, int stock, String imageUrl) {
        this.id = id;
        this.name = name;
        this.description = description;
        this.category = category;
        this.price = price;
        this.stock = stock;
        this.imageUrl = imageUrl;
    }

    // Getters and Setters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getDescription() { return description; }
    public String getCategory() { return category; }
    public double getPrice() { return price; }
    public int getStock() { return stock; }
    public String getImageUrl() { return imageUrl; }
}
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class ProductDAO {
    public List<Product> getProducts(String search, String category, Double minPrice, Double maxPrice) {
        List<Product> products = new ArrayList<>();
        String sql = "SELECT * FROM products WHERE name LIKE ?";

        if (category != null && !category.isEmpty()) {
            sql += " AND category = ?";
        }
        if (minPrice != null) {
            sql += " AND price >= ?";
        }
        if (maxPrice != null) {
            sql += " AND price <= ?";
        }

        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {

            stmt.setString(1, "%" + search + "%");

            int index = 2;
            if (category != null && !category.isEmpty()) {
                stmt.setString(index++, category);
            }
            if (minPrice != null) {
                stmt.setDouble(index++, minPrice);
            }
            if (maxPrice != null) {
                stmt.setDouble(index++, maxPrice);
            }

            ResultSet rs = stmt.executeQuery();
            while (rs.next()) {
                products.add(new Product(
                        rs.getInt("id"),
                        rs.getString("name"),
                        rs.getString("description"),
                        rs.getString("category"),
                        rs.getDouble("price"),
                        rs.getInt("stock"),
                        rs.getString("image_url")
                ));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return products;
    }
}
import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/products")
public class ProductServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String search = request.getParameter("search") != null ? request.getParameter("search") : "";
        String category = request.getParameter("category");
        Double minPrice = request.getParameter("minPrice") != null ? Double.parseDouble(request.getParameter("minPrice")) : null;
        Double maxPrice = request.getParameter("maxPrice") != null ? Double.parseDouble(request.getParameter("maxPrice")) : null;

        ProductDAO productDAO = new ProductDAO();
        List<Product> products = productDAO.getProducts(search, category, minPrice, maxPrice);

        request.setAttribute("products", products);
        request.getRequestDispatcher("product-list.jsp").forward(request, response);
    }
}
<%@ page import="java.util.List" %>
<%@ page import="com.example.Product" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Product Listing</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body>
<div class="container">
    <h2 class="mt-4">Product Listing</h2>

    <!-- Search and Filter Form -->
    <form action="products" method="get" class="row g-3">
        <div class="col-md-3">
            <input type="text" name="search" class="form-control" placeholder="Search..." />
        </div>
        <div class="col-md-3">
            <select name="category" class="form-select">
                <option value="">All Categories</option>
                <option value="Electronics">Electronics</option>
                <option value="Clothing">Clothing</option>
                <option value="Books">Books</option>
            </select>
        </div>
        <div class="col-md-2">
            <input type="number" step="0.01" name="minPrice" class="form-control" placeholder="Min Price" />
        </div>
        <div class="col-md-2">
            <input type="number" step="0.01" name="maxPrice" class="form-control" placeholder="Max Price" />
        </div>
        <div class="col-md-2">
            <button type="submit" class="btn btn-primary">Filter</button>
        </div>
    </form>

    <!-- Product List -->
    <div class="row mt-4">
        <%
            List<Product> products = (List<Product>) request.getAttribute("products");
            if (products != null && !products.isEmpty()) {
                for (Product product : products) {
        %>
        <div class="col-md-4">
            <div class="card mb-3">
                <img src="<%= product.getImageUrl() %>" class="card-img-top" alt="Product Image">
                <div class="card-body">
                    <h5 class="card-title"><%= product.getName() %></h5>
                    <p class="card-text"><%= product.getDescription() %></p>
                    <p class="card-text"><strong>Price:</strong> $<%= product.getPrice() %></p>
                    <p class="card-text"><strong>Category:</strong> <%= product.getCategory() %></p>
                </div>
            </div>
        </div>
        <%
                }
            } else {
        %>
        <p>No products found.</p>
        <%
            }
        %>
    </div>
</div>
</body>
</html>
