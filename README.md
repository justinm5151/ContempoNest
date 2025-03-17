# ContempoNest
Modern room accessories designed to bring style, functionality, and sophistication to any living space.
import React, { useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { ShoppingCart, Trash2 } from "lucide-react";
import { loadStripe } from "@stripe/stripe-js";

const stripePromise = loadStripe("your-stripe-public-key");

const products = [
  {
    id: 1,
    name: "Minimalist LED Lamp",
    price: 59.99,
    image: "/lamp.jpg",
    category: "Lighting",
    description: "A sleek and modern LED lamp perfect for any minimalistic space."
  },
  {
    id: 2,
    name: "Modern Bookshelf",
    price: 129.99,
    image: "/bookshelf.jpg",
    category: "Furniture",
    description: "A stylish bookshelf with a contemporary design to elevate your space."
  },
  {
    id: 3,
    name: "Luxury Velvet Chair",
    price: 199.99,
    image: "/chair.jpg",
    category: "Furniture",
    description: "A luxurious velvet chair that combines comfort with elegance."
  },
];

export default function Store() {
  const [cart, setCart] = useState([]);
  const [filter, setFilter] = useState("All");

  const addToCart = (product) => {
    setCart([...cart, product]);
  };

  const removeFromCart = (index) => {
    setCart(cart.filter((_, i) => i !== index));
  };

  const handleCheckout = async () => {
    const stripe = await stripePromise;
    const response = await fetch("/api/checkout", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ items: cart }),
    });
    const session = await response.json();
    await stripe.redirectToCheckout({ sessionId: session.id });
  };

  const filteredProducts =
    filter === "All" ? products : products.filter((p) => p.category === filter);

  return (
    <div className="p-6 bg-gray-50 min-h-screen">
      <h1 className="text-3xl font-extrabold mb-6 text-center text-gray-800">
        ContempoNest
      </h1>
      <div className="flex justify-center mb-6">
        <label className="mr-2 text-lg">Filter by Category:</label>
        <select
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
          className="border p-2 rounded shadow-sm"
        >
          <option value="All">All</option>
          <option value="Lighting">Lighting</option>
          <option value="Furniture">Furniture</option>
        </select>
      </div>
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        {filteredProducts.map((product) => (
          <Card key={product.id} className="p-6 bg-white shadow-lg rounded-lg">
            <img
              src={product.image}
              alt={product.name}
              className="rounded-lg mb-4 w-full h-48 object-cover"
            />
            <CardContent>
              <h2 className="text-xl font-semibold text-gray-700">
                {product.name}
              </h2>
              <p className="text-gray-500 text-sm mb-2">{product.description}</p>
              <p className="text-gray-500 text-lg font-medium">
                ${product.price.toFixed(2)}
              </p>
              <Button className="mt-4 w-full" onClick={() => addToCart(product)}>
                Add to Cart
              </Button>
            </CardContent>
          </Card>
        ))}
      </div>
      <div className="mt-8 p-6 bg-white shadow-md rounded-lg">
        <h2 className="text-2xl font-bold mb-4 flex items-center text-gray-700">
          <ShoppingCart className="mr-2" /> Cart ({cart.length} items)
        </h2>
        {cart.length > 0 ? (
          <ul>
            {cart.map((item, index) => (
              <li
                key={index}
                className="flex justify-between items-center py-2 border-b"
              >
                <span>{item.name} - ${item.price.toFixed(2)}</span>
                <Button
                  variant="outline"
                  size="sm"
                  onClick={() => removeFromCart(index)}
                  className="text-red-500"
                >
                  <Trash2 className="w-4 h-4" />
                </Button>
              </li>
            ))}
          </ul>
        ) : (
          <p className="text-gray-500">Your cart is empty.</p>
        )}
        <Button
          className="mt-4 w-full bg-green-500 hover:bg-green-600"
          onClick={handleCheckout}
          disabled={cart.length === 0}
        >
          Proceed to Checkout
        </Button>
      </div>
    </div>
  );
}
