# FarmGear
FarmGear 
-- Enable Spatial Search Extension
CREATE EXTENSION IF NOT EXISTS postgis;

-- Users Table (Farmers & Machinery Owners)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    phone_number VARCHAR(15) UNIQUE NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    role VARCHAR(20) CHECK (role IN ('FARMER', 'OWNER', 'BOTH')) DEFAULT 'FARMER',
    location GEOGRAPHY(POINT, 4326),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Equipment Listings
CREATE TABLE equipment (
    id SERIAL PRIMARY KEY,
    owner_id INT REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(150) NOT NULL,
    category VARCHAR(50) NOT NULL, -- e.g., 'Tractor', 'Harvester', 'Drone'
    rate_per_hour NUMERIC(10, 2) NOT NULL,
    rate_per_acre NUMERIC(10, 2),
    operator_provided BOOLEAN DEFAULT TRUE,
    fuel_included BOOLEAN DEFAULT FALSE,
    attachments TEXT,
    is_available BOOLEAN DEFAULT TRUE,
    current_location GEOGRAPHY(POINT, 4326),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Bookings Table
CREATE TABLE bookings (
    id SERIAL PRIMARY KEY,
    equipment_id INT REFERENCES equipment(id),
    renter_id INT REFERENCES users(id),
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL,
    status VARCHAR(20) CHECK (status IN ('PENDING', 'ACCEPTED', 'REJECTED', 'COMPLETED')) DEFAULT 'PENDING',
    payment_status VARCHAR(20) CHECK (payment_status IN ('UNPAID', 'HELD_IN_ESCROW', 'PAID')) DEFAULT 'UNPAID',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Index for Fast Geospatial Proximity Searches
CREATE INDEX idx_equipment_location ON equipment USING GIST (current_location);
