# React + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react/README.md) uses [Babel](https://babeljs.io/) for Fast Refresh
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react-swc) uses [SWC](https://swc.rs/) for Fast Refresh



<!-- for redux file structure to best practice  -->

src/
 ├── api/              # API service files
 │   ├── authService.js
 │   ├── productService.js
 │   └── cartService.js
 ├── features/
 │   ├── authSlice.js
 │   ├── productSlice.js
 │   └── cartSlice.js
 ├── store.js          # Redux store
 └── components/       # Components using Redux state
     ├── Profile.js
     ├── Cart.js
     └── Products.js


<!-- // src/api/authService.js -->
import axios from 'axios';

const API_URL = 'https://example.com/api/auth'; // Base API URL

// Function to handle signup
export const signup = async (userData) => {
  const response = await axios.post(`${API_URL}/signup`, userData);
  return response.data;
};

// Function to handle login
export const signin = async (userData) => {
  const response = await axios.post(`${API_URL}/signin`, userData);
  return response.data;
};

// Function to handle profile fetch
export const fetchProfile = async () => {
  const response = await axios.get(`${API_URL}/profile`);
  return response.data;
};

<!-- // src/api/productService.js -->
import axios from 'axios';

const API_URL = 'https://example.com/api/products'; // Base API URL

// Function to fetch all products
export const fetchProducts = async () => {
  const response = await axios.get(API_URL);
  return response.data;
};

// Function to fetch a single product
export const fetchProductById = async (id) => {
  const response = await axios.get(`${API_URL}/${id}`);
  return response.data;
};


<!-- // src/api/cartService.js -->
import axios from 'axios';

const API_URL = 'https://example.com/api/cart'; // Base API URL

// Function to add item to cart
export const addToCart = async (product) => {
  const response = await axios.post(API_URL, product);
  return response.data;
};

// Function to remove item from cart
export const removeFromCart = async (id) => {
  const response = await axios.delete(`${API_URL}/${id}`);
  return response.data;
};



<!-- // src/features/authSlice.js -->
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { signup, signin, fetchProfile } from '../api/authService';

export const signupUser = createAsyncThunk('auth/signupUser', async (userData) => {
  return await signup(userData); // Call the signup API
});

export const signinUser = createAsyncThunk('auth/signinUser', async (userData) => {
  return await signin(userData); // Call the signin API
});

export const fetchUserProfile = createAsyncThunk('auth/fetchUserProfile', async () => {
  return await fetchProfile(); // Call the profile API
});

const initialState = {
  user: null,
  status: 'idle',
  error: null,
};

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    logout: (state) => {
      state.user = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(signupUser.fulfilled, (state, action) => {
        state.user = action.payload; // Store signed up user data
      })
      .addCase(signinUser.fulfilled, (state, action) => {
        state.user = action.payload; // Store signed in user data
      })
      .addCase(fetchUserProfile.fulfilled, (state, action) => {
        state.user = action.payload; // Store user profile data
      });
  },
});

export const { logout } = authSlice.actions;
export default authSlice.reducer;


<!-- // src/features/productSlice.js -->
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { fetchProducts, fetchProductById } from '../api/productService';

export const getProducts = createAsyncThunk('products/getProducts', async () => {
  return await fetchProducts(); // Call fetch products API
});

export const getProductById = createAsyncThunk('products/getProductById', async (id) => {
  return await fetchProductById(id); // Call fetch product by ID API
});

const initialState = {
  products: [],
  product: null,
  status: 'idle',
  error: null,
};

const productSlice = createSlice({
  name: 'products',
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(getProducts.fulfilled, (state, action) => {
        state.products = action.payload; // Store fetched products
      })
      .addCase(getProductById.fulfilled, (state, action) => {
        state.product = action.payload; // Store single product details
      });
  },
});

export default productSlice.reducer;


<!-- // src/features/cartSlice.js -->
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { addToCart, removeFromCart } from '../api/cartService';

export const addItemToCart = createAsyncThunk('cart/addItemToCart', async (product) => {
  return await addToCart(product); // Call add to cart API
});

export const removeItemFromCart = createAsyncThunk('cart/removeItemFromCart', async (id) => {
  return await removeFromCart(id); // Call remove from cart API
});

const initialState = {
  cart: [],
  status: 'idle',
  error: null,
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(addItemToCart.fulfilled, (state, action) => {
        state.cart.push(action.payload); // Add item to cart
      })
      .addCase(removeItemFromCart.fulfilled, (state, action) => {
        state.cart = state.cart.filter((item) => item.id !== action.payload.id); // Remove item from cart
      });
  },
});

export default cartSlice.reducer;



<!-- // src/store.js -->
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './features/authSlice';
import productReducer from './features/productSlice';
import cartReducer from './features/cartSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    products: productReducer,
    cart: cartReducer,
  },
});


<!-- // src/components/Profile.js -->
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchUserProfile } from '../features/authSlice';

const Profile = () => {
  const dispatch = useDispatch();
  const user = useSelector((state) => state.auth.user);

  useEffect(() => {
    dispatch(fetchUserProfile());
  }, [dispatch]);

  return (
    <div>
      <h1>Profile</h1>
      {user ? <p>{user.name}</p> : <p>Loading...</p>}
    </div>
  );
};

export default Profile;


<!-- // src/components/Products.js -->
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { getProducts } from '../features/productSlice';

const Products = () => {
  const dispatch = useDispatch();
  const products = useSelector((state) => state.products.products);

  useEffect(() => {
    dispatch(getProducts());
  }, [dispatch]);

  return (
    <div>
      <h1>Products</h1>
            <div>
        {products.length > 0 ? (
          products.map((product) => (
            <div key={product.id}>
              <h3>{product.name}</h3>
              <p>{product.description}</p>
              <p>Price: ${product.price}</p>
            </div>
          ))
        ) : (
          <p>Loading products...</p>
        )}
      </div>
    </div>
  );
};

export default Products;


<!-- // src/components/Cart.js -->
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { addItemToCart, removeItemFromCart } from '../features/cartSlice';

const Cart = () => {
  const dispatch = useDispatch();
  const cart = useSelector((state) => state.cart.cart);

  const handleAddItem = (product) => {
    dispatch(addItemToCart(product));
  };

  const handleRemoveItem = (id) => {
    dispatch(removeItemFromCart(id));
  };

  return (
    <div>
      <h1>Cart</h1>
      <div>
        {cart.length > 0 ? (
          cart.map((item) => (
            <div key={item.id}>
              <h3>{item.name}</h3>
              <p>Quantity: {item.quantity}</p>
              <button onClick={() => handleRemoveItem(item.id)}>Remove</button>
            </div>
          ))
        ) : (
          <p>Your cart is empty.</p>
        )}
      </div>
      {/* Example of adding a product to the cart */}
      <button onClick={() => handleAddItem({ id: 1, name: 'Product 1', quantity: 1 })}>
        Add Product 1 to Cart
      </button>
    </div>
  );
};

export default Cart;

#   i m p o r t a n t  
 