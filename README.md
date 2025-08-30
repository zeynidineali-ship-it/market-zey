# market-zey
import React, { useState, useEffect } from 'react';
import { Search, Plus, Heart, Star, User, Menu, X, MessageCircle, Download, ShoppingBag, Store, Shield, Check, Clock, Ban } from 'lucide-react';

const ZeyniShopMarketplace = () => {
  // PWA Installation
  const [deferredPrompt, setDeferredPrompt] = useState(null);
  const [showInstallButton, setShowInstallButton] = useState(false);

  useEffect(() => {
    const handleBeforeInstallPrompt = (e) => {
      e.preventDefault();
      setDeferredPrompt(e);
      setShowInstallButton(true);
    };

    window.addEventListener('beforeinstallprompt', handleBeforeInstallPrompt);

    return () => {
      window.removeEventListener('beforeinstallprompt', handleBeforeInstallPrompt);
    };
  }, []);

  const handleInstallClick = async () => {
    if (deferredPrompt) {
      deferredPrompt.prompt();
      const { outcome } = await deferredPrompt.userChoice;
      if (outcome === 'accepted') {
        setDeferredPrompt(null);
        setShowInstallButton(false);
      }
    }
  };

  // App State
  const [currentView, setCurrentView] = useState('welcome'); // welcome, buyer, seller, admin
  const [showRegistration, setShowRegistration] = useState(false);
  const [registrationType, setRegistrationType] = useState(''); // 'buyer' or 'seller'
  const [isAdmin, setIsAdmin] = useState(false);
  const [showAdminLogin, setShowAdminLogin] = useState(false);
  const [adminPassword, setAdminPassword] = useState("");

  // Users and Products State
  const [users, setUsers] = useState([
    { id: 1, type: 'seller', status: 'approved', name: 'ZeyniShop', surname: 'Admin', whatsapp: '+2290154650905', city: 'Cotonou', address: 'Centre-ville', delivery: true, createdAt: new Date() },
  ]);

  const [products, setProducts] = useState([
    {
      id: 1,
      name: "iPhone 15 Pro Max",
      price: 720000,
      category: "Électronique",
      image: "📱",
      rating: 4.8,
      reviews: 124,
      sellerId: 1,
      sellerName: "ZeyniShop",
      sellerWhatsapp: "+2290154650905",
      description: "Smartphone dernier cri avec appareil photo professionnel",
      favorite: false
    },
    {
      id: 2,
      name: "Sneakers Nike Air Max",
      price: 77000,
      category: "Mode",
      image: "👟",
      rating: 4.6,
      reviews: 89,
      sellerId: 1,
      sellerName: "ZeyniShop",
      sellerWhatsapp: "+2290192278550",
      description: "Chaussures de sport confortables et stylées",
      favorite: true
    }
  ]);

  const [currentUser, setCurrentUser] = useState(null);
  const [searchTerm, setSearchTerm] = useState("");
  const [selectedCategory, setSelectedCategory] = useState("Toutes");
  const [showAddProduct, setShowAddProduct] = useState(false);

  const categories = ["Toutes", "Électronique", "Mode", "Maison", "Sport", "Livres", "Automobile"];

  const [registrationForm, setRegistrationForm] = useState({
    name: "",
    surname: "",
    birthDate: "",
    city: "",
    address: "",
    delivery: false,
    whatsapp: ""
  });

  const [newProduct, setNewProduct] = useState({
    name: "",
    price: "",
    category: "Électronique",
    image: "📦",
    description: ""
  });

  // Welcome Screen Functions
  const handleUserTypeSelection = (type) => {
    setRegistrationType(type);
    setShowRegistration(true);
  };

  // Registration Functions
  const submitRegistration = () => {
    if (!registrationForm.name || !registrationForm.surname || !registrationForm.whatsapp) {
      alert("Veuillez remplir tous les champs obligatoires !");
      return;
    }

    const newUser = {
      id: users.length + 1,
      type: registrationType,
      status: registrationType === 'seller' ? 'pending' : 'approved',
      ...registrationForm,
      createdAt: new Date()
    };

    setUsers([...users, newUser]);

    if (registrationType === 'seller') {
      // Send WhatsApp message to admin for seller approval
      sendSellerRegistrationToAdmin(newUser);
      alert("🎉 Inscription envoyée ! Nous examinerons votre demande de vendeur et vous contacterons bientôt.");
      setCurrentView('welcome');
    } else {
      // Buyer approved immediately
      setCurrentUser(newUser);
      setCurrentView('buyer');
      alert("🎉 Bienvenue sur ZeyniShop ! Vous pouvez maintenant faire vos achats.");
    }

    setShowRegistration(false);
    setRegistrationForm({ name: "", surname: "", birthDate: "", city: "", address: "", delivery: false, whatsapp: "" });
  };

  const sendSellerRegistrationToAdmin = (user) => {
    const message = encodeURIComponent(
      `🏪 NOUVELLE DEMANDE VENDEUR - ZeyniShop\n\n` +
      `👤 Nom: ${user.name} ${user.surname}\n` +
      `📅 Date naissance: ${user.birthDate}\n` +
      `🏙️ Ville: ${user.city}\n` +
      `📍 Adresse: ${user.address}\n` +
      `🚚 Livraison: ${user.delivery ? 'Oui' : 'Non'}\n` +
      `📱 WhatsApp: ${user.whatsapp}\n` +
      `⏰ Demande envoyée: ${new Date().toLocaleDateString()}\n\n` +
      `Voulez-vous approuver ce vendeur sur ZeyniShop ?`
    );
    const whatsappUrl = `https://wa.me/2290154650905?text=${message}`;
    window.open(whatsappUrl, '_blank');
  };

  // Admin Functions
  const adminLogin = () => {
    if (adminPassword === "Zeyni2024") {
      setIsAdmin(true);
      setCurrentView('admin');
      setShowAdminLogin(false);
      setAdminPassword("");
    } else {
      alert("Mot de passe incorrect !");
    }
  };

  const approveUser = (userId) => {
    setUsers(users.map(user => 
      user.id === userId ? { ...user, status: 'approved' } : user
    ));
  };

  const banUser = (userId) => {
    setUsers(users.map(user => 
      user.id === userId ? { ...user, status: 'banned' } : user
    ));
    // Remove products of banned seller
    setProducts(products.filter(product => product.sellerId !== userId));
  };

  // Product Functions
  const openWhatsApp = (whatsappNumber, productName, productPrice, sellerName) => {
    const message = encodeURIComponent(
      `Bonjour ${sellerName}! Je suis intéressé(e) par ce produit sur ZeyniShop:\n\n📱 ${productName}\n💰 Prix: ${productPrice.toLocaleString()} FCFA\n\nPouvons-nous discuter du prix et du paiement?`
    );
    const whatsappUrl = `https://wa.me/${whatsappNumber.replace(/[^0-9]/g, '')}?text=${message}`;
    window.open(whatsappUrl, '_blank');
  };

  const addProduct = () => {
    if (!newProduct.name || !newProduct.price) {
      alert("Veuillez remplir tous les champs obligatoires !");
      return;
    }

    const product = {
      ...newProduct,
      id: products.length + 1,
      price: parseFloat(newProduct.price),
      rating: 4.5,
      reviews: 0,
      sellerId: currentUser?.id || 1,
      sellerName: currentUser?.name || 'ZeyniShop',
      sellerWhatsapp: currentUser?.whatsapp || '+2290154650905',
      favorite: false
    };

    setProducts([...products, product]);
    setNewProduct({ name: "", price: "", category: "Électronique", image: "📦", description: "" });
    setShowAddProduct(false);
    alert("✅ Produit ajouté avec succès !");
  };

  const toggleFavorite = (productId) => {
    setProducts(products.map(product =>
      product.id === productId
        ? { ...product, favorite: !product.favorite }
        : product
    ));
  };

  // Filter products
  const filteredProducts = products.filter(product => {
    const matchesSearch = product.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         product.category.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesCategory = selectedCategory === "Toutes" || product.category === selectedCategory;
    return matchesSearch && matchesCategory;
  });

  const approvedSellers = users.filter(user => user.type === 'seller' && user.status === 'approved');
  const pendingSellers = users.filter(user => user.type === 'seller' && user.status === 'pending');
  const totalBuyers = users.filter(user => user.type === 'buyer').length;

  // Welcome Screen
  if (currentView === 'welcome') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-orange-500 via-orange-600 to-orange-700 flex items-center justify-center p-4">
        <div className="bg-white rounded-3xl shadow-2xl p-8 max-w-md w-full text-center">
          <div className="mb-8">
            <div className="w-20 h-20 bg-gradient-to-r from-orange-500 to-orange-600 rounded-full flex items-center justify-center mx-auto mb-4">
              <span className="text-white font-bold text-3xl">Z</span>
            </div>
            <h1 className="text-3xl font-bold bg-gradient-to-r from-orange-500 to-orange-600 bg-clip-text text-transparent mb-2">
              Bienvenue sur MARKET'ZEY
            </h1>
            <p className="text-gray-600">La marketplace du Bénin 🇧🇯</p>
          </div>

          <div className="space-y-4 mb-8">
            <button
              onClick={() => handleUserTypeSelection('buyer')}
              className="w-full bg-gradient-to-r from-blue-500 to-blue-600 text-white py-4 rounded-xl font-medium hover:shadow-lg transform hover:-translate-y-1 transition-all duration-300 flex items-center justify-center space-x-3"
            >
              <ShoppingBag className="h-6 w-6" />
              <span>Je veux Acheter</span>
            </button>

            <button
              onClick={() => handleUserTypeSelection('seller')}
              className="w-full bg-gradient-to-r from-green-500 to-green-600 text-white py-4 rounded-xl font-medium hover:shadow-lg transform hover:-translate-y-1 transition-all duration-300 flex items-center justify-center space-x-3"
            >
              <Store className="h-6 w-6" />
              <span>Je veux Vendre</span>
            </button>
          </div>

          <div className="pt-4 border-t border-gray-200">
            <button
              onClick={() => setShowAdminLogin(true)}
              className="text-gray-500 hover:text-purple-600 transition-colors text-sm flex items-center justify-center space-x-1"
            >
              <Shield className="h-4 w-4" />
              <span>Administration</span>
            </button>
          </div>
        </div>

        {/* Admin Login Modal */}
        {showAdminLogin && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-2xl max-w-sm w-full p-6">
              <div className="flex justify-between items-center mb-6">
                <h3 className="text-xl font-bold">Connexion Admin</h3>
                <button onClick={() => setShowAdminLogin(false)} className="text-gray-400 hover:text-gray-600">
                  <X className="h-6 w-6" />
                </button>
              </div>
              
              <div className="space-y-4">
                <input
                  type="password"
                  placeholder="Mot de passe admin"
                  className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-orange-500"
                  value={adminPassword}
                  onChange={(e) => setAdminPassword(e.target.value)}
                  onKeyDown={(e) => e.key === 'Enter' && adminLogin()}
                />
                
                <div className="flex space-x-3">
                  <button
                    onClick={() => setShowAdminLogin(false)}
                    className="flex-1 py-3 border border-gray-300 rounded-lg text-gray-700 hover:bg-gray-50"
                  >
                    Annuler
                  </button>
                  <button
                    onClick={adminLogin}
                    className="flex-1 py-3 bg-gradient-to-r from-orange-500 to-orange-600 text-white rounded-lg hover:shadow-lg"
                  >
                    Connexion
                  </button>
                </div>
              </div>
            </div>
          </div>
        )}
      </div>
    );
  }

  // Registration Modal
  if (showRegistration) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-orange-500 via-orange-600 to-orange-700 flex items-center justify-center p-4">
        <div className="bg-white rounded-3xl shadow-2xl p-6 max-w-md w-full">
          <div className="flex justify-between items-center mb-6">
            <h3 className="text-xl font-bold">
              {registrationType === 'buyer' ? '🛒 Inscription Acheteur' : '🏪 Inscription Vendeur'}
            </h3>
            <button onClick={() => setShowRegistration(false)} className="text-gray-400 hover:text-gray-600">
              <X className="h-6 w-6" />
            </button>
          </div>
          
          <div className="space-y-4">
            <input
              type="text"
              placeholder="Nom *"
              className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500"
              value={registrationForm.name}
              onChange={(e) => setRegistrationForm({...registrationForm, name: e.target.value})}
            />
            
            <input
              type="text"
              placeholder="Prénom *"
              className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-orange-500"
              value={registrationForm.surname}
              onChange={(e) => setRegistrationForm({...registrationForm, surname: e.target.value})}
            />
            
            <input
              type="date"
              placeholder="Date de naissance"
              className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-orange-500"
              value={registrationForm.birthDate}
              onChange={(e) => setRegistrationForm({...registrationForm, birthDate: e.target.value})}
            />
            
            <input
              type="text"
              placeholder="Ville de résidence"
              className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-orange-500"
              value={registrationForm.city}
              onChange={(e) => setRegistrationForm({...registrationForm, city: e.target.value})}
            />
            
            <input
              type="text"
              placeholder="Adresse complète"
              className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-orange-500"
              value={registrationForm.address}
              onChange={(e) => setRegistrationForm({...registrationForm, address: e.target.value})}
            />

            {registrationType === 'seller' && (
              <div className="flex items-center space-x-3 p-3 border border-gray-300 rounded-lg">
                <input
                  type="checkbox"
                  id="delivery"
                  checked={registrationForm.delivery}
                  onChange={(e) => setRegistrationForm({...registrationForm, delivery: e.target.checked})}
                  className="w-5 h-5 text-orange-600"
                />
                <label htmlFor="delivery" className="text-gray-700">Je propose la livraison</label>
              </div>
            )}
            
            <input
              type="tel"
              placeholder="Numéro WhatsApp *"
              className="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-orange-500"
              value={registrationForm.whatsapp}
              onChange={(e) => setRegistrationForm({...registrationForm, whatsapp: e.target.value})}
            />
            
            <div className="flex space-x-3">
              <button
                onClick={() => setShowRegistration(false)}
                className="flex-1 py-3 border border-gray-300 rounded-lg text-gray-700 hover:bg-gray-50"
              >
                Annuler
              </button>
              <button
                onClick={submitRegistration}
                className={`flex-1 py-3 text-white rounded-lg hover:shadow-lg ${
                  registrationType === 'buyer' 
                    ? 'bg-gradient-to-r from-blue-500 to-blue-600'
                    : 'bg-gradient-to-r from-green-500 to-green-600'
                }`}
              >
                S'inscrire
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  }

  // Admin Dashboard
  if (currentView === 'admin') {
    return (
      <div className="min-h-screen bg-gray-50">
        <header className="bg-white shadow-lg">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div className="flex justify-between items-center h-16">
              <h1 className="text-2xl font-bold text-orange-600">🛡️ Administration MARKET'ZEY</h1>
              <button
                onClick={() => {
                  setIsAdmin(false);
                  setCurrentView('welcome');
                }}
                className="text-gray-600 hover:text-orange-600 transition-colors"
              >
                Déconnexion
              </button>
            </div>
          </div>
        </header>

        <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          {/* Statistics */}
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div className="bg-white p-6 rounded-xl shadow-md">
              <h3 className="text-lg font-semibold mb-2">👥 Acheteurs</h3>
              <p className="text-3xl font-bold text-blue-600">{totalBuyers}</p>
            </div>
            <div className="bg-white p-6 rounded-xl shadow-md">
              <h3 className="text-lg font-semibold mb-2">🏪 Vendeurs Approuvés</h3>
              <p className="text-3xl font-bold text-green-600">{approvedSellers.length}</p>
            </div>
            <div className="bg-white p-6 rounded-xl shadow-md">
              <h3 className="text-lg font-semibold mb-2">⏳ Demandes en Attente</h3>
              <p className="text-3xl font-bold text-orange-600">{pendingSellers.length}</p>
            </div>
          </div>

          {/* Pending Sellers */}
          {pendingSellers.length > 0 && (
            <div className="bg-white rounded-xl shadow-md p-6 mb-8">
              <h3 className="text-xl font-bold mb-4 text-orange-600">⏳ Demandes de Vendeurs en Attente</h3>
              <div className="space-y-4">
                {pendingSellers.map(seller => (
                  <div key={seller.id} className="border border-orange-200 rounded-lg p-4">
                    <div className="flex justify-between items-start">
                      <div>
                        <h4 className="font-semibold">{seller.name} {seller.surname}</h4>
                        <p className="text-sm text-gray-600">📍 {seller.city} | 📱 {seller.whatsapp}</p>
                        <p className="text-sm text-gray-600">🚚 Livraison: {seller.delivery ? 'Oui' : 'Non'}</p>
                      </div>
                      <div className="flex space-x-2">
                        <button
                          onClick={() => approveUser(seller.id)}
                          className="bg-green-500 text-white px-3 py-1 rounded-lg hover:bg-green-600 text-sm flex items-center space-x-1"
                        >
                          <Check className="h-4 w-4" />
                          <span>Approuver</span>
                        </button>
                        <button
                          onClick={() => banUser(seller.id)}
                          className="bg-red-500 text-white px-3 py-1 rounded-lg hover:bg-red-600 text-sm flex items-center space-x-1"
                        >
                          <Ban className="h-4 w-4" />
                          <span>Refuser</span>
                        </button>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* All Users */}
          <div className="bg-white rounded-xl shadow-md p-6">
            <h3 className="text-xl font-bold mb-4">👥 Tous les Utilisateurs</h3>
            <div className="space-y-3">
              {users.map(user => (
                <div key={user.id} className={`border rounded-lg p-4 ${
                  user.status === 'banned' ? 'border-red-200 bg-red-50' : 
                  user.status === 'pending' ? 'border-orange-200 bg-orange-50' :
                  'border-green-200 bg-green-50'
                }`}>
                  <div className="flex justify-between items-center">
                    <div>
                      <h4 className="font-semibold">
                        {user.name} {user.surname} 
                        <span className={`ml-2 px-2 py-1 rounded-full text-xs ${
                          user.type === 'seller' ? 'bg-green-100 text-green-800' : 'bg-blue-100 text-blue-800'
                        }`}>
                          {user.type === 'seller' ? '🏪 Vendeur' : '🛒 Acheteur'}
                        </span>
                      </h4>
                      <p className="text-sm text-gray-600">📱 {user.whatsapp} | 📍 {user.city}</p>
                    </div>
                    <div className="flex items-center space-x-2">
                      <span className={`px-3 py-1 rounded-full text-sm ${
                        user.status === 'approved' ? 'bg-green-100 text-green-800' :
                        user.status === 'pending' ? 'bg-orange-100 text-orange-800' :
                        'bg-red-100 text-red-800'
                      }`}>
                        {user.status === 'approved' ? '✅ Approuvé' :
                         user.status === 'pending' ? '⏳ En attente' :
                         '🚫 Banni'}
                      </span>
                      {user.status !== 'banned' && user.id !== 1 && (
                        <button
                          onClick={() => banUser(user.id)}
                          className="bg-red-500 text-white px-3 py-1 rounded-lg hover:bg-red-600 text-sm"
                        >
                          Bannir
                        </button>
                      )}
                    </div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        </main>
      </div>
    );
  }

  // Main App (Buyer/Seller View)
  const canAddProducts = currentUser?.type === 'seller' && currentUser?.status === 'approved';

  return (
    <div className="min-h-screen bg-gray-50">
      {/* PWA Install Banner */}
      {showInstallButton && (
        <div className="bg-gradient-to-r from-orange-500 to-orange-600 text-white p-3 text-center relative">
          <div className="flex items-center justify-center space-x-3">
            <Download className="h-5 w-5" />
            <span className="text-sm font-medium">📱 Installer MARKET'ZEY sur votre téléphone</span>
            <button
              onClick={handleInstallClick}
              className="bg-white text-orange-600 px-3 py-1 rounded-full text-xs font-bold hover:bg-gray-100 transition-colors"
            >
              Installer
            </button>
          </div>
          <button
            onClick={() => setShowInstallButton(false)}
            className="absolute right-2 top-1/2 transform -translate-y-1/2 text-white hover:bg-white hover:bg-opacity-20 rounded-full p-1"
          >
            <X className="h-4 w-4" />
          </button>
        </div>
      )}

      {/* Header */}
      <header className="bg-white shadow-lg sticky top-0 z-50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between items-center h-16">
            {/* Logo */}
            <div className="flex items-center space-x-2">
              <div className="w-10 h-10 bg-gradient-to-r from-orange-500 to-orange-600 rounded-lg flex items-center justify-center relative">
                <svg width="20" height="20" viewBox="0 0 24 24" className="text-white">
                  <path 
                    d="M7 18C5.9 18 5.01 18.9 5.01 20S5.9 22 7 22 8.99 21.1 8.99 20 8.1 18 7 18ZM1 2V4H3L6.6 11.59L5.24 14.04C5.09 14.32 5 14.65 5 15C5 16.1 5.9 17 7 17H19V15H7.42C7.28 15 7.17 14.89 7.17 14.75L7.2 14.63L8.1 13H15.55C16.3 13 16.96 12.58 17.3 11.97L20.88 5H5.21L4.27 3H1V2Z" 
                    fill="white"
                    fillOpacity="0.9"
                  />
                  <circle cx="7" cy="20" r="2" fill="white" fillOpacity="0.9"/>
                  <circle cx="17" cy="20" r="2" fill="white" fillOpacity="0.9"/>
                  <text 
                    x="12" 
                    y="11" 
                    fontSize="10" 
                    fontWeight="bold" 
                    fill="#ea580c" 
                    textAnchor="middle" 
                    dominantBaseline="middle"
                    fontFamily="Arial, sans-serif"
                  >
                    Z
                  </text>
                </svg>
              </div>
              <h1 className="text-2xl font-bold bg-gradient-to-r from-orange-500 to-orange-600 bg-clip-text text-transparent">
                MARKET'ZEY
              </h1>
            </div>

            {/* Search Bar */}
            <div className="flex-1 max-w-lg mx-4 sm:mx-8 relative">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 h-5 w-5" />
              <input
                type="text"
                placeholder="Rechercher..."
                className="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-orange-500 focus:border-transparent text-sm sm:text-base"
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
              />
            </div>

            {/* Navigation */}
            <div className="flex items-center space-x-2 sm:space-x-4">
              {canAddProducts && (
                <button
                  onClick={() => setShowAddProduct(true)}
                  className="bg-gradient-to-r from-orange-500 to-orange-600 text-white px-2 sm:px-4 py-2 rounded-lg hover:shadow-lg transition-all duration-300 flex items-center space-x-1 sm:space-x-2 text-xs sm:text-base"
                >
                  <Plus className="h-4 w-4" />
                  <span className="hidden sm:inline">
