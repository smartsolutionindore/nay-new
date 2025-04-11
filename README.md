import { useState } from "react";
import jsPDF from "jspdf";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

const defaultProducts = [
  { id: 1, name: "Product A", price: 1000 },
  { id: 2, name: "Product B", price: 1500 },
  { id: 3, name: "Product C", price: 2000 },
];

export default function QuotationGenerator() {
  const [products, setProducts] = useState(defaultProducts);
  const [selected, setSelected] = useState([]);
  const [newProduct, setNewProduct] = useState({ name: "", price: "" });

  const handleQuantityChange = (id, quantity) => {
    setSelected((prev) => {
      const existing = prev.find((item) => item.id === id);
      if (existing) {
        return prev.map((item) =>
          item.id === id ? { ...item, quantity: Number(quantity) } : item
        );
      } else {
        const product = products.find((p) => p.id === id);
        return [...prev, { ...product, quantity: Number(quantity) }];
      }
    });
  };

  const total = selected.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  const generatePDF = () => {
    const doc = new jsPDF();
    doc.setFontSize(16);
    doc.text("Quotation", 20, 20);
    let y = 30;
    selected.forEach((item, index) => {
      doc.setFontSize(12);
      doc.text(
        `${index + 1}. ${item.name} (x${item.quantity}) - ₹${item.price} = ₹${
          item.price * item.quantity
        }`,
        20,
        y
      );
      y += 10;
    });
    doc.setFontSize(14);
    doc.text(`Total: ₹${total}`, 20, y + 10);
    doc.save("quotation.pdf");
  };

  const handleAddProduct = () => {
    if (!newProduct.name || !newProduct.price) return;
    const newId = products.length ? products[products.length - 1].id + 1 : 1;
    const product = {
      id: newId,
      name: newProduct.name,
      price: Number(newProduct.price),
    };
    setProducts([...products, product]);
    setNewProduct({ name: "", price: "" });
  };

  const handleDeleteProduct = (id) => {
    setProducts(products.filter((p) => p.id !== id));
    setSelected(selected.filter((s) => s.id !== id));
  };

  return (
    <div className="max-w-4xl mx-auto p-4 space-y-8">
      <h1 className="text-3xl font-bold">Product Quotation Website</h1>

      <div className="border p-4 rounded-xl shadow-md">
        <h2 className="text-xl font-semibold mb-4">🛠 Admin Panel</h2>
        <div className="flex gap-4 mb-4">
          <input
            type="text"
            placeholder="Product Name"
            className="border rounded p-2 flex-1"
            value={newProduct.name}
            onChange={(e) =>
              setNewProduct({ ...newProduct, name: e.target.value })
            }
          />
          <input
            type="number"
            placeholder="Price"
            className="border rounded p-2 w-32"
            value={newProduct.price}
            onChange={(e) =>
              setNewProduct({ ...newProduct, price: e.target.value })
            }
          />
          <Button onClick={handleAddProduct}>Add Product</Button>
        </div>
      </div>

      <div className="border p-4 rounded-xl shadow-md">
        <h2 className="text-xl font-semibold mb-4">🛒 Quotation Generator</h2>

        {products.map((product) => (
          <Card key={product.id} className="mb-4">
            <CardContent className="flex justify-between items-center">
              <div>
                <h3 className="text-lg font-medium">{product.name}</h3>
                <p className="text-sm text-gray-600">Price: ₹{product.price}</p>
              </div>
              <div className="flex items-center gap-2">
                <input
                  type="number"
                  min="0"
                  placeholder="Qty"
                  className="w-20 border rounded p-1"
                  onChange={(e) =>
                    handleQuantityChange(product.id, e.target.value)
                  }
                />
                <Button
                  variant="destructive"
                  onClick={() => handleDeleteProduct(product.id)}
                >
                  Delete
                </Button>
              </div>
            </CardContent>
          </Card>
        ))}

        <div className="text-right mt-6">
          <p className="text-xl font-semibold mb-2">Total: ₹{total}</p>
          <Button onClick={generatePDF}>Download Quotation PDF</Button>
        </div>
      </div>
    </div>
  );
}
