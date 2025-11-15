# Epic 2: Website 

Customer-facing web application consuming the API.

---

## Feature 1: Website Foundation

Core website structure, navigation, branding, and foundational UI components.

### User Story #1: Clean Navigation Menu

**As a** customer  
**I want to** have a clean navigation menu  
**So that** I can easily find what I need/want

**Acceptance Criteria:**
- [ ] Navigation bar is visible on all pages and consistently positioned at the top
- [ ] Navigation includes links to: Home, Locations, Menu, Login/Account
- [ ] Navigation is responsive and collapses to hamburger menu on mobile devices (screen width < 768px)
- [ ] Active page is visually indicated in navigation (highlighted/underlined)
- [ ] Navigation is accessible via keyboard (tab navigation works properly)
- [ ] Navigation loads within 100ms without layout shift
- [ ] Hover states provide clear visual feedback on desktop

**Notes / Implementation Details:**
- Use HTML5 semantic `<nav>` element for accessibility
- Implement sticky/fixed navigation for easy access while scrolling
- Use CSS flexbox or grid for responsive layout
- Include ARIA labels for screen reader accessibility
- Consider using a CSS framework like Bootstrap or Tailwind for consistency
- Test across Chrome, Firefox, Safari, and mobile browsers

**Estimated Effort:** Small

### User Story #2: Branded Homepage

**As a** visitor  
**I want to** see a branded homepage  
**So that** I recognize Uncle Joe's identity

**Acceptance Criteria:**
- [ ] Homepage displays Uncle Joe's Coffee Company logo prominently in header
- [ ] Homepage includes hero section with company tagline or welcome message
- [ ] Brand colors are consistent throughout the page (define color palette)
- [ ] Homepage loads in under 2 seconds on standard broadband connection
- [ ] Page includes call-to-action buttons (e.g., "Find a Location", "View Menu")
- [ ] Footer includes company information, contact details, and social media links
- [ ] Page is fully responsive on desktop (1920px), tablet (768px), and mobile (375px) viewports

**Notes / Implementation Details:**
- Design hero section with high-quality coffee imagery
- Use brand fonts consistently (define typography system)
- Include brief "About Uncle Joe's" section highlighting company values
- Optimize images for web (use WebP format with fallbacks)
- Implement lazy loading for below-the-fold images
- Consider adding customer testimonials or featured locations

**Estimated Effort:** Medium

### User Story #3: Quick Links to Menu and Locations

**As a** customer  
**I want** quick links to menu and locations  
**So that** I can navigate fast

**Acceptance Criteria:**
- [ ] Homepage features prominent buttons/cards for "View Menu" and "Find Locations"
- [ ] Buttons are large enough for easy clicking/tapping (minimum 44x44px touch target)
- [ ] Links navigate to correct pages (/menu and /locations)
- [ ] Buttons have clear, action-oriented text labels
- [ ] Hover/focus states provide visual feedback before clicking
- [ ] Buttons are accessible via keyboard navigation (Enter key activates)
- [ ] Links work consistently across all devices and browsers

**Notes / Implementation Details:**
- Position quick links prominently in hero section or immediately below
- Use contrasting colors for buttons to make them stand out
- Implement smooth page transitions or loading states
- Consider adding icons to buttons (map pin for locations, coffee cup for menu)
- Ensure buttons maintain visibility above the fold on mobile devices
- Track button clicks with analytics for user behavior insights

**Estimated Effort:** Small

### User Story #4: Modern UI Template

**As a** PM  
**I want to** have a modern UI template  
**So that** the site feels up to date

**Acceptance Criteria:**
- [ ] Website uses contemporary design patterns (clean layout, ample whitespace, modern typography)
- [ ] Design system includes consistent spacing, sizing, and color schemes
- [ ] UI components follow current web design trends (rounded corners, subtle shadows, smooth animations)
- [ ] Website passes accessibility standards (WCAG 2.1 AA minimum)
- [ ] Design is mobile-first and fully responsive
- [ ] Template includes reusable components (buttons, cards, forms, modals)
- [ ] Stakeholders approve design mockups/wireframes before implementation

**Notes / Implementation Details:**
- Use a CSS framework (Bootstrap, Tailwind, or Material UI) for consistency
- Create a style guide/design system document
- Implement dark mode support (optional, future enhancement)
- Use CSS variables for easy theme customization
- Include microinteractions (button hover effects, smooth scrolling, fade-ins)
- Ensure consistent component behavior across all pages
- Document component usage for development team

**Estimated Effort:** Large

### User Story #5: Error Boundary Handling

**As a** frontend developer  
**I want to** implement error boundary handling  
**So that** the app gracefully handles failures

**Acceptance Criteria:**
- [ ] Application displays user-friendly error messages when components fail to load
- [ ] Errors are caught and logged to console for debugging purposes
- [ ] Error boundaries prevent entire app from crashing due to component errors
- [ ] Fallback UI displays when errors occur (e.g., "Something went wrong" message with refresh option)
- [ ] Network errors (API failures) display appropriate messages (e.g., "Unable to load data. Please try again.")
- [ ] 404 pages display for invalid routes with navigation back to home
- [ ] Error states are tested in development and staging environments

**Notes / Implementation Details:**
- Implement React Error Boundaries (if using React) or equivalent error handling
- Create reusable ErrorFallback component for consistent error UI
- Log errors to monitoring service (e.g., Sentry, LogRocket) for production tracking
- Display different error messages for network vs application errors
- Include "Retry" or "Go Home" buttons in error states
- Test error scenarios: API failures, invalid data, network timeouts
- Ensure errors don't expose sensitive information or stack traces to users

**Estimated Effort:** Medium

---

## Feature 2: Locations & Menu Display

Pages for browsing store locations and viewing menu items with search and filtering capabilities.

### User Story #1: Browse All Store Locations

**As a** customer  
**I want to** browse all store locations  
**So that** I can pick one to visit

**Acceptance Criteria:**
- [ ] Locations page displays all Uncle Joe's store locations
- [ ] Each location card shows store_name, address, city, state, phone, and hours
- [ ] Locations are displayed in a grid or list view (user can toggle between views)
- [ ] Page includes search/filter functionality (search by city, state, or zip code)
- [ ] Clicking a location card shows detailed view with map and additional information
- [ ] Page is responsive and displays properly on mobile, tablet, and desktop
- [ ] Page loads all locations in under 2 seconds

**Notes / Implementation Details:**
- Fetch location data from GET /api/v1/locations endpoint
- Display locations in card/tile format with consistent styling
- Implement client-side filtering for better performance
- Consider pagination or infinite scroll for large location lists (500 locations)
- Show store status (open/closed) based on current time and hours_open/hours_close
- Include distance from user's location (if geolocation permission granted)
- Add "Get Directions" link to open in Google Maps or Apple Maps

**Estimated Effort:** Medium

### User Story #2: Retrieve List of Store Locations

**As a** customer  
**I want to** retrieve a list of store locations  
**So that** I can find the nearest coffee shop

**Acceptance Criteria:**
- [ ] Page displays locations sorted by proximity to user's current location (if location permission granted)
- [ ] Each location shows distance from user in miles or kilometers
- [ ] Locations without user location permission are sorted alphabetically by city
- [ ] "Use My Location" button prompts user for geolocation permission
- [ ] Location permission denied displays message: "Enable location to find nearby stores"
- [ ] Distance calculation is accurate within 0.1 miles
- [ ] Nearest location is visually highlighted or marked as "Closest to You"

**Notes / Implementation Details:**
- Use browser Geolocation API to get user's current position
- Calculate distances using Haversine formula or similar geospatial calculation
- Handle geolocation errors gracefully (permission denied, timeout, unavailable)
- Cache user's location for session to avoid repeated permission prompts
- Display skeleton loaders while fetching location data
- Allow manual location entry via zip code or city if geolocation unavailable
- Consider using map view to show user position and nearby stores

**Estimated Effort:** Large

### User Story #3: Search for Stores by Zip Code

**As a** customer  
**I want to** search for stores by zip code  
**So that** I can find the nearest location

**Acceptance Criteria:**
- [ ] Search bar accepts 5-digit zip code input
- [ ] Search results display stores within entered zip code area
- [ ] Invalid zip codes display error message: "Please enter a valid 5-digit zip code"
- [ ] Results show stores sorted by distance from zip code center
- [ ] Search executes on button click or Enter key press
- [ ] "Clear" button resets search and shows all locations
- [ ] Search is case-insensitive and handles leading/trailing spaces

**Notes / Implementation Details:**
- Validate zip code format using regex (5 digits only)
- Filter locations by matching zip_code field from BigQuery
- Consider expanding search to include nearby zip codes within radius (e.g., 10 miles)
- Implement autocomplete for zip code suggestions (optional)
- Display count of results found (e.g., "5 locations found in 47906")
- Persist search query in URL parameters for shareable links
- Add search icon for visual clarity

**Estimated Effort:** Medium

### User Story #4: View Menu Visually

**As a** customer  
**I want to** view the menu visually  
**So that** I can see available items

**Acceptance Criteria:**
- [ ] Menu page displays all menu items organized by category (Coffee, Food, Merchandise)
- [ ] Each menu item shows name, description, price, and availability status
- [ ] Items include images (if available) for visual appeal
- [ ] Unavailable items are visually distinguished (grayed out or marked "Out of Stock")
- [ ] Menu is responsive and displays well on all device sizes
- [ ] Page loads in under 2 seconds
- [ ] Menu items are displayed in consistent card/tile format

**Notes / Implementation Details:**
- Fetch menu data from GET /api/v1/menu endpoint
- Group items by category field for organized display
- Use grid layout for menu items (3-4 columns on desktop, 1-2 on mobile)
- Display placeholder image if item image is unavailable
- Format prices consistently ($X.XX format)
- Show dietary information icons (vegan, gluten-free) if applicable
- Implement lazy loading for item images
- Consider adding "Add to Favorites" functionality (future enhancement)

**Estimated Effort:** Medium

### User Story #5: View Detailed Product Information

**As a** customer  
**I want to** view detailed product information (nutritional facts, availability, and photos)  
**So that** I can identify any allergens before choosing an item

**Acceptance Criteria:**
- [ ] Clicking a menu item opens detailed view (modal or separate page)
- [ ] Detail view displays full-size product image, name, and description
- [ ] Nutritional information is clearly displayed (calories, fat, protein, allergens)
- [ ] Dietary restrictions are indicated (vegan, gluten-free, dairy-free, nut-free)
- [ ] Availability status is shown (Available, Out of Stock, Seasonal)
- [ ] Price and size options are clearly listed
- [ ] "Close" or "Back" button returns to menu page

**Notes / Implementation Details:**
- Extend menu_items data model to include nutritional_info and allergens fields
- Display allergen warnings prominently (e.g., "Contains: Milk, Soy, Nuts")
- Use icons or badges for dietary information for quick scanning
- Include high-quality product photos (optimize for web)
- Implement modal component for smooth user experience
- Allow swiping between items in modal on mobile devices
- Add "Share" button to share product via social media or link (optional)
- Ensure modal is accessible (keyboard navigation, screen reader support)

**Estimated Effort:** Large

### User Story #6: Filter Menu Items

**As a** customer  
**I want to** be able to filter menu items  
**So that** I can find what I want to purchase

**Acceptance Criteria:**
- [ ] Filter options include: Category (Coffee, Food, Merchandise), Dietary restrictions (Vegan, Gluten-free), Price range
- [ ] Multiple filters can be applied simultaneously (e.g., Coffee + Vegan)
- [ ] Filtered results update immediately without page reload
- [ ] Filter count displays number of items shown (e.g., "Showing 12 of 50 items")
- [ ] "Clear Filters" button resets all filters to default state
- [ ] No results message displays when filters return empty set
- [ ] Filters are visually indicated as active/inactive

**Notes / Implementation Details:**
- Implement client-side filtering for performance (filter fetched menu data)
- Use checkboxes or toggle buttons for filter options
- Implement price range slider for price filtering
- Persist filter state in URL parameters for shareable filtered views
- Show active filters as removable tags/chips
- Consider adding search bar for keyword filtering (search by item name)
- Test filter combinations to ensure accurate results
- Optimize filtering algorithm for large menu datasets

**Estimated Effort:** Large

---

## Feature 3: Member Features

Member login, account dashboard, order history, and loyalty program features.

### User Story #1: View Loyalty Points and Rewards

**As a** member  
**I want to** view my loyalty points and the rewards  
**So that** I understand what benefits are available to me

**Acceptance Criteria:**
- [ ] Account dashboard displays current loyalty points balance prominently
- [ ] Dashboard shows points progress toward next reward tier or milestone
- [ ] Available rewards are listed with points required for redemption
- [ ] Points history shows recent points earned and redeemed
- [ ] Dashboard displays member since date and total lifetime points
- [ ] Rewards are visually organized (e.g., "50 points = Free Coffee", "200 points = Free Pastry")
- [ ] Page updates in real-time when points balance changes

**Notes / Implementation Details:**
- Fetch loyalty_points from GET /api/v1/members/{member_id} endpoint
- Display points with visual indicator (progress bar or circular gauge)
- Show points expiration date if applicable
- Implement rewards catalog section showing redeemable items
- Use icons or images for rewards to enhance visual appeal
- Calculate and display points needed for next reward
- Consider adding points earning rules explanation (e.g., "Earn 1 point per $1 spent")
- Include "Learn More" link to full rewards program details

**Estimated Effort:** Medium

### User Story #2: See Past Orders

**As a** member  
**I want to** see my past orders  
**So that** I can track my spending

**Acceptance Criteria:**
- [ ] Order history page displays list of past orders sorted by date (most recent first)
- [ ] Each order shows order_id, date, location, total amount, and status
- [ ] Clicking an order expands or navigates to detailed order view
- [ ] Order details include itemized list with quantities and prices
- [ ] Order history includes search/filter by date range, location, or status
- [ ] Total spending is calculated and displayed (e.g., "Total Spent: $342.50")
- [ ] Page supports pagination (20 orders per page)

**Notes / Implementation Details:**
- Fetch order data from GET /api/v1/orders/history/{member_id} endpoint
- Display orders in card or table format depending on screen size
- Include order status badge (Completed, Pending, Cancelled)
- Show thumbnail images of ordered items for visual context
- Calculate spending statistics (total spent, average order value, most frequent location)
- Implement "Reorder" button to quickly place same order again
- Handle empty state when user has no order history ("No orders yet. Start exploring our menu!")
- Ensure only authenticated user can view their own order history (JWT validation)

**Estimated Effort:** Large

### User Story #3: View Member Profile

**As a** member  
**I want to** view my profile  
**So that** I can see all of my information in one place

**Acceptance Criteria:**
- [ ] Profile page displays member_id, email, first_name, last_name, phone, and join_date
- [ ] Preferred location is shown with option to change
- [ ] Profile includes "Edit Profile" button to update information
- [ ] Current loyalty points balance is displayed on profile page
- [ ] Account creation date (join_date) is formatted in readable format (e.g., "Member Since: January 2024")
- [ ] Profile page is accessible only when user is logged in
- [ ] Changes to profile auto-save or require explicit "Save Changes" action

**Notes / Implementation Details:**
- Fetch profile data from GET /api/v1/members/{member_id} endpoint using authenticated member_id
- Display fields in organized sections (Personal Info, Contact Info, Preferences)
- Implement edit mode with form validation for profile updates (future enhancement)
- Show profile completion percentage if some fields are optional
- Include "Change Password" link (future enhancement)
- Display account security settings (email notifications, privacy preferences)
- Ensure password_hash is never displayed in frontend
- Add "Delete Account" option with confirmation modal (optional)

**Estimated Effort:** Medium

### User Story #4: See Most Frequently Ordered Items

**As a** member  
**I want to** see my most frequently ordered items based on my order history  
**So that** I can quickly reorder an item

**Acceptance Criteria:**
- [ ] Dashboard displays "Your Favorites" or "Frequently Ordered" section
- [ ] Section shows top 3-5 most frequently ordered items with images
- [ ] Each item displays name, last ordered date, and total times ordered
- [ ] "Reorder" button allows quick reorder of favorite item
- [ ] Favorites are calculated from order history (most ordered items)
- [ ] Section updates dynamically as order history grows
- [ ] Empty state displays when user has no order history ("Order something to see your favorites!")

**Notes / Implementation Details:**
- Analyze order history to calculate item frequency (count occurrences of each item_id)
- Fetch item details from menu API to display name and image
- Sort items by frequency (descending) and display top results
- Implement "Reorder" functionality to add item to cart (Note: project scope is view-only, so redirect to menu or show item details)
- Consider time-based weighting (recent orders weighted more heavily)
- Display frequency metric (e.g., "Ordered 8 times")
- Cache calculated favorites to improve performance
- Update favorites calculation periodically or on-demand

**Estimated Effort:** Large

### User Story #5: User Login

**As a** user  
**I want to** be able to log in  
**So that** I can access my account information

**Acceptance Criteria:**
- [ ] Login page includes email and password input fields
- [ ] Form validates email format before submission
- [ ] Successful login redirects to account dashboard
- [ ] Failed login displays error message: "Invalid email or password"
- [ ] Login form includes "Remember Me" checkbox for persistent sessions
- [ ] "Forgot Password" link is available (future enhancement)
- [ ] JWT token is stored securely (localStorage or sessionStorage)
- [ ] User session persists across page refreshes

**Notes / Implementation Details:**
- Submit credentials to POST /api/v1/auth/login endpoint
- Validate email format using regex on client side
- Store JWT token in localStorage for "Remember Me", sessionStorage otherwise
- Set token expiration handling (auto-logout when token expires)
- Redirect to original requested page after successful login (if user was redirected to login)
- Implement client-side validation for better UX (check empty fields before submit)
- Include password visibility toggle (eye icon to show/hide password)
- Protect against CSRF attacks (use anti-CSRF tokens if applicable)
- Display loading spinner during authentication request

**Estimated Effort:** Medium