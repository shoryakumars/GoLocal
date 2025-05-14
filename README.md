# GoLocal
import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Search, ShoppingCart, MapPin, Bell, Store } from "lucide-react";

export default function GoLocalUIPrototype() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [loginData, setLoginData] = useState({ username: "", password: "" });
  const [showLocationDialog, setShowLocationDialog] = useState(false);
  const [location, setLocation] = useState("");
  const [manualLocation, setManualLocation] = useState("");
  const [suggestedLocations, setSuggestedLocations] = useState([]);
  const [searchQuery, setSearchQuery] = useState("");
  const [notification, setNotification] = useState(null);
  const [cart, setCart] = useState([]);
  const [showCart, setShowCart] = useState(false);
  const [showVendors, setShowVendors] = useState(false);
  const [categoryWiseVendors, setCategoryWiseVendors] = useState({});
  const [searchResults, setSearchResults] = useState(null);

  const vendors = [
    { id: 1, name: "Bunty All in One Shop", category: "Groceries", distance: "1.3km" },
    { id: 2, name: "Mother Dairy", category: "Dairy", distance: "2km" },
    { id: 3, name: "VegeMart", category: "Vegetables", distance: "1.5km" },
    { id: 4, name: "JunkBites", category: "Snacks", distance: "5km" }
  ];

  const suggestedItems = ["Vegetables", "Groceries", "Food Items", "Fast Food", "Medicines"];

  const productsByCategory = {
    Dairy: [
      { name: "Milk", price: 25, image: "milk.jpg" },
      { name: "Cheese", price: 80, image: "cheese.jpg" },
      { name: "Butter", price: 90, image: "butter.jpg" },
      { name: "Yogurt", price: 40, image: "yogurt.jpg" }
    ],
    Vegetables: [
      { name: "Spinach", price: 15, image: "spinach.jpg" },
      { name: "Carrots", price: 20, image: "carrots.jpg" },
      { name: "Lettuce", price: 25, image: "lettuce.jpg" },
      { name: "Cabbage", price: 18, image: "cabbage.jpg" },
      { name: "Tomatoes", price: 22, image: "tomatoes.jpg" }
    ],
    Fruits: [
      { name: "Apples", price: 40, image: "apples.jpg" },
      { name: "Bananas", price: 30, image: "bananas.jpg" },
      { name: "Oranges", price: 35, image: "oranges.jpg" }
    ],
    Snacks: [
      { name: "Chips", price: 35, image: "chips.jpg" },
      { name: "Cookies", price: 45, image: "cookies.jpg" },
      { name: "Juice", price: 60, image: "juice.jpg" }
    ],
    Essentials: [
      { name: "Toothpaste", price: 50, image: "toothpaste.jpg" },
      { name: "Soap", price: 20, image: "soap.jpg" },
      { name: "Shampoo", price: 70, image: "shampoo.jpg" }
    ],
    Toys: [
      { name: "Toy Car", price: 120, image: "toycar.jpg" },
      { name: "Teddy Bear", price: 200, image: "teddy.jpg" },
      { name: "Building Blocks", price: 150, image: "blocks.jpg" }
    ]
  };

  const allProducts = Object.entries(productsByCategory).flatMap(([cat, items]) =>
    items.map((item) => ({ ...item, category: cat }))
  );

  const handleSearch = () => {
    if (searchQuery.trim() !== "") {
      const result = allProducts.find(p => p.name.toLowerCase() === searchQuery.toLowerCase());
      setSearchResults(result ? [result] : []);
    } else {
      setSearchResults(null);
    }
  };

  const handleLogin = () => {
    if (loginData.username === "GOLOCAL" && loginData.password === "PLACEMENT") {
      setIsLoggedIn(true);
      setShowLocationDialog(true);
    } else {
      alert("Invalid credentials");
    }
  };

  const handleLocation = () => {
    if (manualLocation.trim() !== "") {
      setLocation(manualLocation);
    } else {
      navigator.geolocation.getCurrentPosition((position) => {
        setLocation(`Lat: ${position.coords.latitude}, Lon: ${position.coords.longitude}`);
      });
    }
    setShowLocationDialog(false);
  };

  const handleManualLocationChange = (value) => {
    setManualLocation(value);
    setSuggestedLocations(["New Delhi", "Noida", "Gurugram", "Ghaziabad"].filter(loc =>
      loc.toLowerCase().includes(value.toLowerCase())
    ));
  };

  const addToCart = (product) => {
    const index = cart.findIndex((item) => item.name === product.name);
    if (index > -1) {
      const updatedCart = [...cart];
      updatedCart[index].quantity += 1;
      setCart(updatedCart);
    } else {
      setCart([...cart, { ...product, quantity: 1 }]);
    }
  };

  const removeFromCart = (product) => {
    const updatedCart = cart.map(item =>
      item.name === product.name ? { ...item, quantity: item.quantity - 1 } : item
    ).filter(item => item.quantity > 0);
    setCart(updatedCart);
  };

  const totalPrice = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);

  const placeOrder = () => {
    setShowCart(false);
    const vendorsForCategories = {};
    cart.forEach((item) => {
      const vendor = vendors.find(v => v.category === item.category);
      if (vendor) vendorsForCategories[item.category] = vendor;
    });
    setCategoryWiseVendors(vendorsForCategories);
    setShowVendors(true);
  };

  const renderProductCard = (product, key) => {
    const inCart = cart.find(item => item.name === product.name);
    return (
      <Card key={key} className="p-4 bg-white shadow-lg rounded-lg">
        <CardContent>
          <img src={product.image} alt={product.name} className="h-24 w-full object-cover rounded mb-2" />
          <h4 className="text-md font-semibold text-gray-800">{product.name}</h4>
          <p className="text-sm text-gray-500">Price: ₹{product.price}</p>
          {inCart ? (
            <div className="flex justify-between items-center mt-2">
              <Button onClick={() => removeFromCart(product)}>-</Button>
              <span>{inCart.quantity}</span>
              <Button onClick={() => addToCart(product)}>+</Button>
            </div>
          ) : (
            <Button className="mt-2 w-full bg-green-600 text-white" onClick={() => addToCart(product)}>Add to Cart</Button>
          )}
        </CardContent>
      </Card>
    );
  };

  if (!isLoggedIn) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-gray-100">
        <Card className="p-6 w-96 shadow-lg">
          <h2 className="text-2xl font-bold mb-4">Login to GoLocal</h2>
          <Input placeholder="Username" value={loginData.username} onChange={(e) => setLoginData({ ...loginData, username: e.target.value })} className="mb-2" />
          <Input type="password" placeholder="Password" value={loginData.password} onChange={(e) => setLoginData({ ...loginData, password: e.target.value })} className="mb-4" />
          <Button className="w-full bg-blue-600 text-white" onClick={handleLogin}>Login</Button>
        </Card>
      </div>
    );
  }

  return (
    <div className="p-6 bg-gray-100 min-h-screen">
      {showLocationDialog && (
        <Dialog open={showLocationDialog} onOpenChange={setShowLocationDialog}>
          <DialogContent>
            <DialogHeader>
              <DialogTitle>Select Your Location</DialogTitle>
            </DialogHeader>
            <Input placeholder="Enter location manually" value={manualLocation} onChange={(e) => handleManualLocationChange(e.target.value)} className="mb-2" />
            <ul>
              {suggestedLocations.map((loc, idx) => (
                <li key={idx} className="cursor-pointer text-blue-600 hover:underline" onClick={() => setManualLocation(loc)}>{loc}</li>
              ))}
            </ul>
            <Button onClick={handleLocation} className="mt-2">Use This Location</Button>
            <Button onClick={() => navigator.geolocation.getCurrentPosition(pos => setLocation(`Lat: ${pos.coords.latitude}, Lon: ${pos.coords.longitude}`))} className="mt-2 bg-green-500 text-white">Use Current Location</Button>
          </DialogContent>
        </Dialog>
      )}

      <div className="flex items-center justify-between bg-blue-500 p-4 rounded-lg shadow-lg">
        <h1 className="text-3xl font-bold text-white">GoLocal - {location}</h1>
        <div className="flex space-x-3">
          <Input placeholder="Search for products..." className="w-64 p-2 rounded-md" value={searchQuery} onChange={(e) => setSearchQuery(e.target.value)} />
          <Button variant="outline" onClick={handleSearch} className="bg-yellow-400 text-white font-semibold">
            <Search className="w-5 h-5" />
          </Button>
          <Button onClick={() => setShowCart(true)} className="bg-green-500 text-white font-semibold">
            <ShoppingCart className="w-5 h-5" /> ({cart.length})
          </Button>
        </div>
      </div>

      {searchResults && (
        <div className="mt-4 bg-white p-4 rounded-lg shadow-md">
          <Button className="mb-2 bg-gray-300 text-black" onClick={() => setSearchResults(null)}>Back</Button>
          <h3 className="text-xl font-semibold text-gray-800 mb-2">Search Result</h3>
          {searchResults.length === 0 ? <p>No matching product found.</p> : (
            <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
              {searchResults.map((product, index) => renderProductCard(product, `search-${index}`))}
            </div>
          )}
        </div>
      )}

      {!searchResults && Object.entries(productsByCategory).map(([category, items]) => (
        <div key={category} className="mt-6">
          <h3 className="text-xl font-semibold text-gray-800 mb-2">{category}</h3>
          <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
            {items.map((product, index) => renderProductCard(product, `${category}-${index}`))}
          </div>
        </div>
      ))}

      <Dialog open={showCart} onOpenChange={setShowCart}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Your Cart</DialogTitle>
          </DialogHeader>
          {cart.length === 0 ? (
            <p>No items in cart.</p>
          ) : (
            <ul className="space-y-2">
              {cart.map((item, index) => (
                <li key={`${item.name}-${index}`} className="flex justify-between">
                  <span>{item.name} x{item.quantity}</span>
                  <span>₹{item.price * item.quantity}</span>
                </li>
              ))}
            </ul>
          )}
          <div className="mt-4 font-semibold">Total Price: ₹{totalPrice}</div>
          <Button className="mt-2 bg-blue-600 text-white w-full" onClick={placeOrder}>Place Order</Button>
        </DialogContent>
      </Dialog>

      <Dialog open={showVendors} onOpenChange={setShowVendors}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Nearby Vendors Based on Your Cart</DialogTitle>
          </DialogHeader>
          <ul className="mt-2 space-y-2">
            {Object.entries(categoryWiseVendors).map(([category, vendor]) => (
              <li key={category} className="p-2 bg-gray-100 rounded-md">
                <strong>{vendor.name}</strong> - {vendor.category} ({vendor.distance})
              </li>
            ))}
            {vendors.map((vendor) => (
              <li key={vendor.id} className="p-2 bg-gray-50 rounded-md">
                <strong>{vendor.name}</strong> - {vendor.category} ({vendor.distance})
              </li>
            ))}
          </ul>
          <div className="mt-4 text-green-600 font-bold">Notification sent to vendor(s).</div>
        </DialogContent>
      </Dialog>
    </div>
  );
}
