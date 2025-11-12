# Final-POE
import * as React from 'react';
import { useState, createContext, useContext, useEffect, useMemo } from 'react';
import {
  ImageBackground, Text, SafeAreaView,
  StyleSheet, TextInput, ScrollView, View, Alert, TouchableOpacity, Image, Animated
} from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// ------------------ CONTEXTS ------------------
const CartContext = createContext();
const MenuContext = createContext();

// ------------------ NAVIGATION ------------------
const Stack = createNativeStackNavigator();

// ------------------ SMALL HELPERS ------------------
const calcAverage = (items) => {
  if (!items || items.length === 0) return 0;
  const sum = items.reduce((s, it) => s + (Number(it.price) || 0), 0);
  return +(sum / items.length).toFixed(2);
};

// ------------------ HOME SCREEN ------------------
function HomeScreen({ navigation }) {
  return (
    <SafeAreaView style={styles.container}>
      <ImageBackground
        source={require('./assets/Background.png')}
        style={styles.background}
        resizeMode="cover"
      >
        <View style={styles.logoContainer}>
          <Image source={require('./assets/TasteTap.webp')} style={styles.logo} resizeMode="contain" />
          <Text style={styles.logoText}>Taste Tap</Text>
        </View>

        <Text style={styles.h2}>Welcome to Taste Tap</Text>

        <TouchableOpacity style={styles.mainButton} onPress={() => navigation.navigate('Menu')}>
          <Text style={styles.buttonText}>Proceed</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.mainButton, { backgroundColor: '#333' }]} onPress={() => navigation.navigate('AdminLogin')}>
          <Text style={styles.buttonText}>Admin Login</Text>
        </TouchableOpacity>

        <Text style={styles.slogan}>Savor Every Sip, Relish Every Bite</Text>
      </ImageBackground>
    </SafeAreaView>
  );
}

// ------------------ ADMIN LOGIN ------------------
function AdminLoginScreen({ navigation }) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = () => {
    if (username === 'Admin' && password === '12345') {
      navigation.navigate('AdminDashboard');
    } else {
      Alert.alert('Error', 'Invalid credentials');
    }
  };

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.h1}>Admin Login</Text>
      <TextInput style={styles.input} placeholder="Username" value={username} onChangeText={setUsername} />
      <TextInput style={styles.input} placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      <TouchableOpacity style={styles.mainButton} onPress={handleLogin}>
        <Text style={styles.buttonText}>Login</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

// ------------------ ADMIN DASHBOARD ------------------
function AdminDashboard({ navigation }) {
  const { menus, setMenus } = useContext(MenuContext);
  const [selectedMenu, setSelectedMenu] = useState('Main');

  const items = menus[selectedMenu] || [];
  const average = useMemo(() => calcAverage(items), [items]);

  const removeItem = (id) => {
    setMenus(prev => ({ ...prev, [selectedMenu]: prev[selectedMenu].filter(i => i.id !== id) }));
  };

  return (
    <SafeAreaView style={styles.container}>
      <TouchableOpacity style={[styles.cardButton, styles.logout]} onPress={() => navigation.navigate('Home')}>
        <Text style={styles.buttonText}>Logout</Text>
      </TouchableOpacity>

      <Text style={styles.h1}>Admin Dashboard</Text>

      <View style={{ flexDirection: 'row', justifyContent: 'space-around', marginVertical: 10 }}>
        {['Main', 'Dessert', 'Special'].map(menu => (
          <TouchableOpacity key={menu} style={[styles.secondaryButton, selectedMenu === menu && { backgroundColor: '#FF0000' }]} onPress={() => setSelectedMenu(menu)}>
            <Text style={styles.buttonText}>{menu} Menu</Text>
          </TouchableOpacity>
        ))}
      </View>

      <Text style={styles.h2}>Items: {items.length} • Average: R{average}</Text>

      <ScrollView style={{ marginVertical: 10 }}>
        {items.map(item => (
          <View key={item.id} style={{ flexDirection: 'row', justifyContent: 'space-between', padding: 10, borderBottomWidth: 1, borderColor: '#ccc' }}>
            <Text>{item.title} - R{item.price}</Text>
            <TouchableOpacity style={[styles.cardButton, { padding: 5 }]} onPress={() => removeItem(item.id)}>
              <Text style={styles.buttonText}>Remove</Text>
            </TouchableOpacity>
          </View>
        ))}
      </ScrollView>

      <View style={{ flexDirection: 'row', justifyContent: 'space-between' }}>
        <TouchableOpacity style={[styles.mainButton, { flex: 0.48 }]} onPress={() => navigation.navigate('AdminAddItem', { selectedMenu })}>
          <Text style={styles.buttonText}>Add New Item</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.mainButton, { flex: 0.48, backgroundColor: '#333' }]} onPress={() => navigation.navigate('Menu')}>
          <Text style={styles.buttonText}>Go to User Side</Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  );
}

// ------------------ ADMIN ADD ITEM (SEPARATE SCREEN) ------------------
function AdminAddItemScreen({ navigation, route }) {
  const { menus, setMenus } = useContext(MenuContext);
  const selectedMenuFromRoute = route.params?.selectedMenu || 'Main';
  const [selectedMenu, setSelectedMenu] = useState(selectedMenuFromRoute);
  const [newItem, setNewItem] = useState('');
  const [newPrice, setNewPrice] = useState('');

  const addMenuItem = () => {
    if (!newItem || !newPrice) return Alert.alert('Validation', 'Please provide both name and price');
    const item = { id: Date.now(), title: newItem.trim(), price: parseFloat(newPrice) };
    setMenus(prev => ({ ...prev, [selectedMenu]: [...prev[selectedMenu], item] }));
    setNewItem('');
    setNewPrice('');
    Alert.alert('Success', 'Item added');
  };

  return (
    <SafeAreaView style={styles.container}>
      <TouchableOpacity style={[styles.cardButton, styles.logout]} onPress={() => navigation.goBack()}>
        <Text style={styles.buttonText}>Back</Text>
      </TouchableOpacity>

      <Text style={styles.h1}>Add Menu Item</Text>

      <View style={{ flexDirection: 'row', justifyContent: 'space-around', marginVertical: 10 }}>
        {['Main', 'Dessert', 'Special'].map(menu => (
          <TouchableOpacity key={menu} style={[styles.secondaryButton, selectedMenu === menu && { backgroundColor: '#FF0000' }]} onPress={() => setSelectedMenu(menu)}>
            <Text style={styles.buttonText}>{menu}</Text>
          </TouchableOpacity>
        ))}
      </View>

      <TextInput style={styles.input} placeholder="New Item Name" value={newItem} onChangeText={setNewItem} />
      <TextInput style={styles.input} placeholder="Price" value={newPrice} onChangeText={setNewPrice} keyboardType="numeric" />

      <TouchableOpacity style={styles.mainButton} onPress={addMenuItem}>
        <Text style={styles.buttonText}>Add</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

// ------------------ DETAILS PAGE ------------------
function DetailsScreen({ navigation }) {
  const [name, setName] = useState('');
  const [surname, setSurname] = useState('');

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.h1}>Enter Your Details</Text>
      <TextInput style={styles.input} placeholder="Enter your name" value={name} onChangeText={setName} />
      <TextInput style={styles.input} placeholder="Enter your surname" value={surname} onChangeText={setSurname} />
      <TouchableOpacity style={styles.mainButton} onPress={() => navigation.navigate('Checkout', { name, surname })}>
        <Text style={styles.buttonText}>Continue to Checkout</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

// ------------------ MENU PAGE (USER) ------------------
function MenuScreen({ navigation }) {
  const { cartItems, addToCart, clearCart } = useContext(CartContext);
  const { menus } = useContext(MenuContext);

  const [currentMenu, setCurrentMenu] = useState('Main');
  const [fadeAnim] = useState(new Animated.Value(0));
  const [cartScale] = useState(new Animated.Value(1));

  const animateCards = () => {
    fadeAnim.setValue(0);
    Animated.timing(fadeAnim, { toValue: 1, duration: 400, useNativeDriver: true }).start();
  };

  const animateCart = () => {
    Animated.sequence([
      Animated.timing(cartScale, { toValue: 1.3, duration: 120, useNativeDriver: true }),
      Animated.timing(cartScale, { toValue: 1, duration: 120, useNativeDriver: true }),
    ]).start();
  };

  useEffect(() => { animateCards(); }, [currentMenu, menus]);

  const items = menus[currentMenu] || [];
  const average = useMemo(() => calcAverage(items), [items]);

  const handleAddToCart = (item) => { addToCart(item); animateCart(); };

  const handleCashOut = () => {
    if (cartItems.length === 0) {
      Alert.alert('Cart Empty', 'Please add at least one item.');
      return;
    }
    navigation.navigate('Details');
  };

  return (
    <SafeAreaView style={styles.container}>
      <TouchableOpacity style={[styles.cardButton, styles.logout]} onPress={() => navigation.navigate('Home')}>
        <Text style={styles.buttonText}>Logout</Text>
      </TouchableOpacity>

      <ScrollView>
        <Text style={styles.h1}>Menu</Text>
        <Text style={styles.h2}>Showing: {currentMenu} • Items: {items.length} • Avg: R{average}</Text>

        <View style={{ flexDirection: 'row', justifyContent: 'space-around', marginVertical: 10 }}>
          {['Main', 'Dessert', 'Special'].map(menu => (
            <TouchableOpacity key={menu} style={[styles.secondaryButton, currentMenu === menu && { backgroundColor: '#FF0000' }]} onPress={() => setCurrentMenu(menu)}>
              <Text style={styles.buttonText}>{menu}</Text>
            </TouchableOpacity>
          ))}
        </View>

        <View style={styles.grid}>
          {items.map(item => (
            <Animated.View key={item.id} style={[styles.card, { opacity: fadeAnim }]}>
              <Text style={styles.foodTitle}>{item.title}</Text>
              <Text style={styles.price}>R{item.price}</Text>
              <TouchableOpacity style={styles.cardButton} onPress={() => handleAddToCart(item)}>
                <Text style={styles.buttonText}>Add to Cart</Text>
              </TouchableOpacity>
            </Animated.View>
          ))}
        </View>

        <View style={{ flexDirection: 'row', justifyContent: 'space-between', marginVertical: 10 }}>
          <TouchableOpacity style={[styles.mainButton, { flex: 0.48 }]} onPress={handleCashOut}>
            <Text style={styles.buttonText}>Cash Out</Text>
          </TouchableOpacity>

          <TouchableOpacity style={[styles.mainButton, { flex: 0.48, backgroundColor: '#333' }]} onPress={clearCart}>
            <Text style={styles.buttonText}>Clear Cart</Text>
          </TouchableOpacity>
        </View>

        <TouchableOpacity style={[styles.mainButton, { backgroundColor: '#0066CC' }]} onPress={() => navigation.navigate('GuestFilter')}>
          <Text style={styles.buttonText}>Filter As Guest</Text>
        </TouchableOpacity>

      </ScrollView>

      <View style={styles.bottomBar}>
        <Animated.Text style={[styles.cartText, { transform: [{ scale: cartScale }] }]}>
          🛒 Cart: {cartItems.length}
        </Animated.Text>
      </View>
    </SafeAreaView>
  );
}

// ------------------ GUEST FILTER SCREEN ------------------
function GuestFilterScreen({ navigation }) {
  const { menus } = useContext(MenuContext);
  const [filter, setFilter] = useState('Main');

  const items = menus[filter] || [];
  const average = useMemo(() => calcAverage(items), [items]);

  return (
    <SafeAreaView style={styles.container}>
      <TouchableOpacity style={[styles.cardButton, styles.logout]} onPress={() => navigation.goBack()}>
        <Text style={styles.buttonText}>Back</Text>
      </TouchableOpacity>

      <Text style={styles.h1}>Guest Filter</Text>
      <Text style={styles.h2}>Filtering: {filter} • Items: {items.length} • Avg: R{average}</Text>

      <View style={{ flexDirection: 'row', justifyContent: 'space-around', marginVertical: 10 }}>
        {['Main', 'Dessert', 'Special'].map(menu => (
          <TouchableOpacity key={menu} style={[styles.secondaryButton, filter === menu && { backgroundColor: '#FF0000' }]} onPress={() => setFilter(menu)}>
            <Text style={styles.buttonText}>{menu}</Text>
          </TouchableOpacity>
        ))}
      </View>

      <ScrollView>
        {items.map(it => (
          <View key={it.id} style={{ padding: 12, borderBottomWidth: 1, borderColor: '#eee' }}>
            <Text style={{ fontWeight: '600' }}>{it.title}</Text>
            <Text>R{it.price}</Text>
          </View>
        ))}
      </ScrollView>
    </SafeAreaView>
  );
}

// ------------------ CHECKOUT PAGE ------------------
function CheckoutScreen({ route }) {
  const { cartItems, clearCart } = useContext(CartContext);
  const total = cartItems.reduce((sum, item) => sum + item.price, 0);
  const name = route.params?.name;
  const surname = route.params?.surname;

  const handlePlaceOrder = () => {
    const orderNumber = Math.floor(Math.random() * 1000000);
    Alert.alert('Order Placed', `Your order number is #${orderNumber}. Please pay at the till.`, [{ text: 'OK', onPress: clearCart }]);
  };

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.h1}>Order Summary</Text>
      {name || surname ? (
        <Text style={styles.p1}>For: {name} {surname}</Text>
      ) : null}
      <ScrollView>
        {cartItems.map((item, index) => (
          <View key={index} style={{ marginVertical: 5 }}>
            <Text>{item.title} - R{item.price}</Text>
          </View>
        ))}
        <Text style={{ marginTop: 10, fontWeight: 'bold', fontSize: 18 }}>Total: R{total}</Text>

        <View style={{ flexDirection: 'row', justifyContent: 'space-between', marginTop: 20 }}>
          <TouchableOpacity style={[styles.mainButton, { flex: 0.48 }]} onPress={handlePlaceOrder}>
            <Text style={styles.buttonText}>Place Order</Text>
          </TouchableOpacity>

          <TouchableOpacity style={[styles.mainButton, { flex: 0.48, backgroundColor: '#333' }]} onPress={clearCart}>
            <Text style={styles.buttonText}>Clear Cart</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

// ------------------ APP ------------------
export default function App() {
  const [cartItems, setCartItems] = useState([]);
  const [menus, setMenus] = useState({
    Main: [
      { id: 1, title: 'Burger', price: 100 },
      { id: 2, title: 'Salad', price: 80 },
      { id: 3, title: 'Pizza', price: 120 },
      { id: 4, title: 'Pasta', price: 110 },
      { id: 5, title: 'Fries', price: 60 },
      { id: 6, title: 'Drink', price: 40 },
    ],
    Dessert: [
      { id: 11, title: 'Ice Cream', price: 50 },
      { id: 12, title: 'Donut', price: 40 },
      { id: 13, title: 'Cookie', price: 30 },
    ],
    Special: [
      { id: 21, title: 'Steak', price: 200 },
      { id: 22, title: 'Sushi', price: 180 },
    ],
  });

  const addToCart = (item) => setCartItems(prev => [...prev, item]);
  const removeFromCart = (id) => setCartItems(prev => prev.filter((_, index) => index !== id));
  const clearCart = () => setCartItems([]);

  return (
    <MenuContext.Provider value={{ menus, setMenus }}>
      <CartContext.Provider value={{ cartItems, addToCart, removeFromCart, clearCart }}>
        <NavigationContainer>
          <Stack.Navigator initialRouteName="Home">
            <Stack.Screen name="Home" component={HomeScreen} />
            <Stack.Screen name="AdminLogin" component={AdminLoginScreen} />
            <Stack.Screen name="AdminDashboard" component={AdminDashboard} />
            <Stack.Screen name="AdminAddItem" component={AdminAddItemScreen} />
            <Stack.Screen name="Details" component={DetailsScreen} />
            <Stack.Screen name="Menu" component={MenuScreen} />
            <Stack.Screen name="GuestFilter" component={GuestFilterScreen} />
            <Stack.Screen name="Checkout" component={CheckoutScreen} />
          </Stack.Navigator>
        </NavigationContainer>
      </CartContext.Provider>
    </MenuContext.Provider>
  );
}

// ------------------ STYLES ------------------
const styles = StyleSheet.create({
  container: { flex: 1, padding: 8, backgroundColor: '#FDFDFD' },
  background: { flex: 1 },
  logoContainer: { flexDirection: 'row', alignItems: 'center', margin: 20 },
  logo: { width: 60, height: 60, marginRight: 10 },
  logoText: { fontSize: 40, fontWeight: 'bold', color: '#FF0000' },
  h1: { margin: 10, fontSize: 25, fontWeight: 'bold', color: '#FF0000' },
  h2: { margin: 10, fontSize: 20, fontWeight: 'bold', color: '#FF0000' },
  p1: { margin: 10, fontSize: 18, color: 'black' },
  slogan: { margin: 10, fontSize: 20, fontWeight: 'bold', color: 'black', fontFamily: 'Snell Roundhand' },
  input: { borderWidth: 1, borderColor: '#ccc', padding: 12, marginBottom: 15, borderRadius: 8, fontSize: 16 },
  mainButton: { backgroundColor: '#FF0000', padding: 12, borderRadius: 8, alignItems: 'center', marginVertical: 8 },
  secondaryButton: { backgroundColor: '#FFA500', padding: 12, borderRadius: 8, alignItems: 'center', flex: 0.3 },
  cardButton: { backgroundColor: '#FF4500', padding: 8, borderRadius: 6, alignItems: 'center', marginTop: 5 },
  buttonText: { color: '#fff', fontWeight: 'bold', fontSize: 16 },
  grid: { flexDirection: 'row', flexWrap: 'wrap', justifyContent: 'space-between' },
  card: { backgroundColor: '#fff', padding: 15, marginVertical: 10, borderRadius: 12, shadowColor: '#000', shadowOpacity: 0.15, shadowOffset: { width: 0, height: 4 }, shadowRadius: 6, elevation: 6, width: '48%' },
  foodTitle: { fontSize: 18, fontWeight: 'bold', marginBottom: 5 },
  price: { fontSize: 16, fontWeight: '600', marginVertical: 5, color: '#FF0000' },
  cashOutButton: { marginVertical: 15 },
  bottomBar: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', padding: 12, borderTopWidth: 1, borderColor: '#ddd', backgroundColor: '#f7f7f7' },
  cartText: { fontSize: 18, fontWeight: 'bold', color: '#FF0000' },
  logout: { position: 'absolute', top: 10, right: 10, zIndex: 10 }
});
