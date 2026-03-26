<!DOCTYPE html>
<html lang="hi" data-theme="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Zpioxw - Local Shop Bazar</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=DM+Sans:wght@400;500;700&display=swap');

        :root {
            --bg:#071020; --bg2:#0c2233; --bg3:#0f2d44;
            --card:#0c2233; --border:#1e4060;
            --text:#e8f2fe; --text2:#94c0e8; --muted:#8ab0c8;
            --sky:#0ea5e9; --sky2:#38bdf8; --sky3:#7dd3fc;
            --ok:#22c55e; --warn:#f59e0b; --err:#ef4444; --r:14px;
        }

        [data-theme="light"] {
            --bg:#f8fafc; --bg2:#f0f9ff; --bg3:#e0f2fe;
            --card:#ffffff; --border:#bae6fd;
            --text:#0f172a; --text2:#1e3a5f; --muted:#475569;
            --sky:#0284c7; --sky3:#0369a1;
        }

        * { margin:0; padding:0; box-sizing:border-box; -webkit-tap-highlight-color:transparent; font-family:'DM+Sans', sans-serif; }
        body { background:var(--bg); color:var(--text); line-height:1.6; overflow-x:hidden; transition:0.3s; padding-bottom: 80px; }

        /* HEADER */
        header { position:sticky; top:0; z-index:100; background:rgba(7, 16, 32, 0.8); backdrop-filter:blur(15px); border-bottom:1px solid var(--border); padding:12px 16px; display:flex; justify-content:space-between; align-items:center; }
        .logo { font-family:'Playfair Display', serif; font-size:24px; font-weight:900; background:linear-gradient(to right, var(--sky2), #fff); -webkit-background-clip:text; -webkit-text-fill-color:transparent; }

        /* CATEGORY SECTION */
        .category-container {
            padding: 15px 16px;
            overflow-x: auto;
            display: flex;
            gap: 10px;
            white-space: nowrap;
            scrollbar-width: none;
        }
        .category-container::-webkit-scrollbar { display: none; }

        .cat-chip {
            padding: 8px 18px;
            background: var(--bg2);
            border: 1px solid var(--border);
            border-radius: 50px;
            color: var(--text2);
            font-size: 14px;
            font-weight: 500;
            cursor: pointer;
            transition: 0.3s;
        }
        .cat-chip.active {
            background: var(--sky);
            color: white;
            border-color: var(--sky);
            box-shadow: 0 4px 12px rgba(14, 165, 233, 0.3);
        }

        /* SEARCH BAR */
        .search-box { padding: 10px 16px; }
        .search-box input {
            width: 100%;
            padding: 12px 15px;
            background: var(--bg3);
            border: 1px solid var(--border);
            border-radius: var(--r);
            color: var(--text);
            outline: none;
        }

        /* PRODUCT GRID */
        .product-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 12px;
            padding: 16px;
        }
        .product-card {
            background: var(--card);
            border: 1px solid var(--border);
            border-radius: var(--r);
            padding: 12px;
            text-align: center;
            animation: fadeIn 0.5s ease;
        }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        .product-card img { width: 100%; border-radius: 10px; margin-bottom: 8px; background: #1a2a3a; }
        .product-card h3 { font-size: 14px; margin: 5px 0; color: var(--text); height: 40px; overflow: hidden; }
        .price { color: var(--sky2); font-weight: bold; font-size: 16px; }
        
        .add-btn {
            width: 100%;
            padding: 8px;
            margin-top: 10px;
            background: var(--sky);
            border: none;
            border-radius: 8px;
            color: white;
            font-weight: bold;
            cursor: pointer;
        }

        /* FOOTER NAV */
        .footer-nav {
            position: fixed;
            bottom: 0;
            width: 100%;
            background: var(--bg2);
            display: flex;
            justify-content: space-around;
            padding: 15px;
            border-top: 1px solid var(--border);
            z-index: 100;
        }
        .nav-item { display: flex; flex-direction: column; align-items: center; font-size: 12px; color: var(--muted); cursor: pointer; }
        .nav-item.active { color: var(--sky2); }
    </style>
</head>
<body>

<header>
    <div class="logo">Zpioxw</div>
    <div id="theme-toggle" style="font-size: 20px; cursor: pointer;">🌙</div>
</header>

<div class="search-box">
    <input type="text" id="searchInput" placeholder="Search products, groceries..." onkeyup="searchProduct()">
</div>

<div class="category-container" id="categoryList">
    <div class="cat-chip active" onclick="filterCat('all', this)">All Items</div>
    <div class="cat-chip" onclick="filterCat('Grocery', this)">Grocery</div>
    <div class="cat-chip" onclick="filterCat('Spices', this)">Spices</div>
    <div class="cat-chip" onclick="filterCat('Snacks', this)">Snacks</div>
    <div class="cat-chip" onclick="filterCat('Cold Drinks', this)">Cold Drinks</div>
</div>

<div class="product-grid" id="productDisplay">
    <p style="grid-column: span 2; text-align: center; padding: 20px; color: var(--muted);">Loading products...</p>
</div>

<nav class="footer-nav">
    <div class="nav-item active"><span>🏠</span><span>Home</span></div>
    <div class="nav-item"><span>🛒</span><span>Cart</span></div>
    <div class="nav-item"><span>📦</span><span>Orders</span></div>
    <div class="nav-item"><span>👤</span><span>Profile</span></div>
</nav>

<script>
    // Theme Toggle Logic
    const themeBtn = document.getElementById('theme-toggle');
    themeBtn.onclick = () => {
        const current = document.documentElement.getAttribute('data-theme');
        const next = current === 'dark' ? 'light' : 'dark';
        document.documentElement.setAttribute('data-theme', next);
        themeBtn.innerText = next === 'dark' ? '🌙' : '☀️';
    };

    // Dummy Data (Jab tak aap Excel connect na karein)
    let allProducts = [
        { name: "Turmeric Powder", price: 45, cat: "Spices", img: "https://via.placeholder.com/150?text=Turmeric" },
        { name: "Basmati Rice", price: 120, cat: "Grocery", img: "https://via.placeholder.com/150?text=Rice" },
        { name: "Potato Chips", price: 20, cat: "Snacks", img: "https://via.placeholder.com/150?text=Chips" },
        { name: "Coca Cola", price: 40, cat: "Cold Drinks", img: "https://via.placeholder.com/150?text=Coke" },
        { name: "Red Chili", price: 55, cat: "Spices", img: "https://via.placeholder.com/150?text=Chili" },
        { name: "Cooking Oil", price: 180, cat: "Grocery", img: "https://via.placeholder.com/150?text=Oil" }
    ];

    // Function to Show Products
    function displayProducts(products) {
        const container = document.getElementById('productDisplay');
        container.innerHTML = "";
        
        if(products.length === 0) {
            container.innerHTML = `<p style="grid-column: span 2; text-align: center; color: var(--err);">No items found!</p>`;
            return;
        }

        products.forEach(p => {
            container.innerHTML += `
                <div class="product-card">
                    <img src="${p.img}" alt="${p.name}">
                    <h3>${p.name}</h3>
                    <p class="price">₹${p.price}</p>
                    <button class="add-btn">Add to Cart</button>
                </div>
            `;
        });
    }

    // Category Filter Logic
    function filterCat(category, element) {
        // Active class change karein
        document.querySelectorAll('.cat-chip').forEach(c => c.classList.remove('active'));
        element.classList.add('active');

        if(category === 'all') {
            displayProducts(allProducts);
        } else {
            const filtered = allProducts.filter(p => p.cat === category);
            displayProducts(filtered);
        }
    }

    // Search Logic
    function searchProduct() {
        const term = document.getElementById('searchInput').value.toLowerCase();
        const searched = allProducts.filter(p => p.name.toLowerCase().includes(term));
        displayProducts(searched);
    }

    // Initial Load
    window.onload = () => displayProducts(allProducts);

</script>

</body>
</html>
