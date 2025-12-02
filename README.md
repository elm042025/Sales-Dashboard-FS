# Sales Dashboard Full-Stack

A real-time sales tracking dashboard built with React, Vite, and Supabase. Track sales deals, visualize team performance, and manage sales representatives with live data updates.

## 🌟 Features

- **Real-time Updates**: Live dashboard updates using Supabase Realtime
- **User Authentication**: Secure sign-up/sign-in with Supabase Auth
- **Role-based Access**: Admin and Sales Rep account types with different permissions
- **Interactive Charts**: Visual representation of sales data using react-charts
- **Sales Management**: Add and track sales deals per representative
- **Responsive Design**: Works seamlessly on desktop and mobile devices

## 🏗️ Tech Stack

- **Frontend**: React 19, React Router v7
- **Build Tool**: Vite 6
- **Backend/Database**: Supabase (PostgreSQL + Auth + Realtime)
- **Styling**: CSS
- **Charts**: react-charts
- **Linting**: ESLint

## 📋 Prerequisites

Before you begin, ensure you have the following installed:

- [Node.js](https://nodejs.org/) (v18 or higher recommended)
- [npm](https://www.npmjs.com/) (comes with Node.js)
- A [Supabase](https://supabase.com/) account (free tier works)

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/elm042025/Sales-Dashboard-FS
cd Sales-Dashboard-FS
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Set Up Supabase

#### Create a Supabase Project

1. Go to [Supabase](https://supabase.com/) and sign in
2. Click "New Project"
3. Fill in your project details and wait for setup to complete

#### Create Database Tables

Run the following SQL in your Supabase SQL Editor:

```sql
-- Create user_profiles table
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  account_type TEXT NOT NULL CHECK (account_type IN ('admin', 'rep')),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create sales_deals table
CREATE TABLE sales_deals (
  id SERIAL PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  value INTEGER NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE sales_deals ENABLE ROW LEVEL SECURITY;

-- Create policies for user_profiles
CREATE POLICY "Users can view all profiles"
  ON user_profiles FOR SELECT
  TO authenticated
  USING (true);

CREATE POLICY "Users can insert their own profile"
  ON user_profiles FOR INSERT
  TO authenticated
  WITH CHECK (auth.uid() = id);

-- Create policies for sales_deals
CREATE POLICY "Users can view all deals"
  ON sales_deals FOR SELECT
  TO authenticated
  USING (true);

CREATE POLICY "Admins can insert any deal"
  ON sales_deals FOR INSERT
  TO authenticated
  WITH CHECK (
    EXISTS (
      SELECT 1 FROM user_profiles
      WHERE id = auth.uid() AND account_type = 'admin'
    )
  );

CREATE POLICY "Reps can insert their own deals"
  ON sales_deals FOR INSERT
  TO authenticated
  WITH CHECK (user_id = auth.uid());
```

#### Set Up Database Trigger

Create a trigger to automatically insert user profile on signup:

```sql
-- Function to handle new user signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.user_profiles (id, name, account_type)
  VALUES (
    NEW.id,
    NEW.raw_user_meta_data->>'name',
    NEW.raw_user_meta_data->>'account_type'
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger the function on user signup
CREATE OR REPLACE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

#### Enable Realtime

1. Go to your Supabase project settings
2. Navigate to Database → Replication
3. Enable replication for the `sales_deals` table

### 4. Configure Environment Variables

Create a `.env` file in the project root:

```bash
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_KEY=your_supabase_anon_key
```

**How to find these values:**

1. Go to your Supabase project settings
2. Click on "Settings" → "API"
3. Copy the "Project URL" and "anon/public" key

### 5. Run the Application

```bash
npm run dev
```

The app will start on `http://localhost:5173` (or the next available port if 5173 is in use).

## 📖 How to Use

### Creating an Account

1. Navigate to the app in your browser
2. Click "Sign up today!"
3. Fill in your name, email, and password
4. Select your role:
   - **Admin**: Can add deals for any sales rep
   - **Sales Rep**: Can only add their own deals
5. Click "Sign Up"

### Adding Sales Deals

**As an Admin:**

1. Sign in with your admin account
2. Use the form on the dashboard
3. Select a sales rep from the dropdown
4. Enter the deal amount
5. Click "Add Deal"

**As a Sales Rep:**

1. Sign in with your sales rep account
2. Use the form on the dashboard
3. Your name is pre-filled
4. Enter the deal amount
5. Click "Add Deal"

### Viewing Dashboard

- The chart displays total sales per representative for the current quarter
- Data updates in real-time when new deals are added
- All authenticated users can view the dashboard

## 🔐 Security Notes

- Never commit your `.env` file to version control
- The `.env` file is already in `.gitignore`
- Use environment-specific values for production deployments
- Row Level Security (RLS) policies protect your data

## 🛠️ Available Scripts

```bash
npm run dev      # Start development server
npm run build    # Build for production
npm run preview  # Preview production build
npm run lint     # Run ESLint
```

## 📁 Project Structure

```
Sales-Dashboard-FS/
├── src/
│   ├── components/
│   │   ├── Form.jsx              # Form to add sales deals
│   │   ├── Header.jsx            # App header with sign out
│   │   ├── ProtectedRoute.jsx    # Auth route protection
│   │   ├── Signin.jsx            # Sign in page
│   │   └── Signup.jsx            # Sign up page
│   ├── context/
│   │   └── AuthContext.jsx       # Authentication context
│   ├── routes/
│   │   ├── Dashboard.jsx         # Main dashboard with chart
│   │   └── RootRedirect.jsx      # Root route handler
│   ├── index.css                 # Global styles
│   ├── main.jsx                  # App entry point
│   ├── router.jsx                # Route configuration
│   └── supabase-client.js        # Supabase client setup
├── public/                       # Static assets
├── .env                          # Environment variables (not in repo)
├── .env.example                  # Example env file
├── .gitignore                    # Git ignore rules
├── index.html                    # HTML entry point
├── package.json                  # Dependencies and scripts
├── vite.config.js                # Vite configuration
└── README.md                     # This file
```

## 🐛 Troubleshooting

### Port Already in Use

If you see "Port 5173 is in use", Vite will automatically use the next available port (e.g., 5174).

### Supabase Connection Issues

- Verify your `.env` file contains the correct URL and key
- Check that your Supabase project is running
- Ensure environment variables start with `VITE_` prefix

### Authentication Not Working

- Confirm email confirmation is disabled in Supabase Auth settings (for development)
- Check database triggers are created correctly
- Verify RLS policies are enabled

### Real-time Updates Not Working

- Ensure Replication is enabled for `sales_deals` table in Supabase
- Check browser console for connection errors

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 💬 Support

If you have any questions or run into issues, please open an issue on GitHub.

---

Built with ❤️ using React and Supabase
