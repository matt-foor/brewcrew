
# Epic 1: API
Core backend services and data access layer.
---

## Feature E1-F4: Members API
Provides secure, read-only endpoints to retrieve member information, authentication data, and loyalty points from BigQuery.

---

### User Story E1-F4-US1: Member GET Endpoint Creation

**As a** developer  
**I want to** create member GET endpoint  
**So that** users can log in and retrieve their profile information

**Acceptance Criteria:**
- [ ] GET `/api/v1/members/{member_id}` endpoint returns member profile data
- [ ] Endpoint returns 200 status code with valid member data when member exists
- [ ] Endpoint returns 404 status code when member_id does not exist
- [ ] Response includes member_id, email, first_name, last_name, join_date, loyalty_points, phone, and preferred_location_id
- [ ] Endpoint validates JWT token before returning member data
- [ ] Password_hash field is excluded from response for security
- [ ] API documentation is generated/updated with endpoint details

**Notes / Implementation Details:**
- Use FastAPI path parameters for member_id
- Query BigQuery `uncle_joes_coffee.members` table
- Implement JWT token validation decorator/dependency
- Return Pydantic model for type safety and automatic documentation
- Consider implementing field-level permissions if needed

**Estimated Effort:** Medium

---

### User Story E1-F4-US2: User Login to Website

**As a** user  
**I want to** be able to log in to the website  
**So that** I can see my profile information

**Acceptance Criteria:**
- [ ] Login form accepts email and password
- [ ] Login request performs read-only password verification using password_hash
- [ ] No member data is created or updated during login
- [ ] JWT tokens issued without modifying database
- [ ] Successful login redirects user to account dashboard
- [ ] Failed login displays clear error message (invalid credentials)
- [ ] User session persists across page refreshes
- [ ] Logout functionality clears user session
- [ ] Login form validates email format before submission
- [ ] Password field masks input characters

**Notes / Implementation Details:**
- Frontend implementation using HTML/CSS/JavaScript
- Store JWT token in localStorage or sessionStorage
- Include "Remember Me" checkbox for extended sessions (optional)
- Implement client-side validation for better UX
- Handle network errors gracefully with user-friendly messages

**Estimated Effort:** Medium

---

### User Story E1-F4-US3: Authentication API Client Interface

**As an** API client  
**I want to** be able to provide user login information and receive information on authentication  
**So that** I can facilitate user logins

**Acceptance Criteria:**
- [ ] POST `/api/v1/auth/login` endpoint accepts email and password in request body
- [ ] Endpoint returns JWT token on successful authentication
- [ ] Endpoint returns 401 status code with error message for invalid credentials
- [ ] JWT token includes member_id, email, and expiration timestamp in payload
- [ ] Token expiration is set to configurable duration (default: 24 hours)
- [ ] Endpoint implements rate limiting to prevent brute force attacks (e.g., 5 attempts per minute)
- [ ] Password is hashed and compared securely (bcrypt or similar)

**Notes / Implementation Details:**
- Use PyJWT library for token generation
- Hash passwords using bcrypt with appropriate salt rounds
- Query BigQuery to validate credentials against members table
- Return token format: `{"access_token": "jwt_string", "token_type": "bearer"}`
- Consider implementing refresh token mechanism for enhanced security
- Log failed login attempts for security monitoring

**Estimated Effort:** Large

---

### User Story E1-F4-US4: View Loyalty Points

**As a** user  
**I want to** be able to view my loyalty points on my account  
**So that** I can keep track of my rewards

**Acceptance Criteria:**
- [ ] Account dashboard displays current loyalty points balance prominently
- [ ] Loyalty points are retrieved from authenticated user's profile
- [ ] Points display updates in real-time when user navigates to account page
- [ ] UI clearly indicates points balance (e.g., "You have 1,250 points")
- [ ] Error handling displays message if points cannot be loaded
- [ ] Points display is consistent across desktop and mobile views

**Notes / Implementation Details:**
- Fetch loyalty_points from GET `/api/v1/members/{member_id}` endpoint
- Use authenticated member_id from JWT token
- Display points with thousand separators for readability (e.g., 1,250)
- Consider showing points history or redemption options in future iterations
- Cache points data briefly to reduce API calls

**Estimated Effort:** Small

---
## 
---

## Feature E1-F5: Orders API
Offers read-only access to order history and enriched order details by combining MongoDB and BigQuery data.

---

### User Story E1-F5-US1: Order History Retrieval Endpoint

**As a** developer  
**I want to** create order history retrieval endpoint  
**So that** members can view past purchases

**Acceptance Criteria:**
- [ ] GET `/api/v1/orders/history/{member_id}` endpoint returns list of orders
- [ ] Endpoint returns orders sorted by order_date (most recent first)
- [ ] Response includes order_id, location_id, order_date, items array, total_amount, and status
- [ ] Response includes item name, category, price, and dietary_info from BigQuery
- [ ] API performs backend-side join (MongoDB + BigQuery)
- [ ] Endpoint supports pagination with query parameters (e.g., `?page=1&limit=10`)
- [ ] Endpoint validates JWT token and ensures member_id matches authenticated user
- [ ] Returns 200 status code with empty array if no orders exist
- [ ] Returns 401 if user attempts to access another member's order history

**Notes / Implementation Details:**
- Query MongoDB `uncle_joes_coffee.orders` collection
- Filter by member_id (add member_id field to order documents if not present)
- Default limit: 20 orders per page
- Return total count in response metadata for pagination UI
- Consider adding date range filters (e.g., `?start_date=2025-01-01&end_date=2025-12-31`)

**Estimated Effort:** Medium

---

### User Story E1-F5-US2: Order Status Tracking Endpoint

**As a** developer  
**I want to** implement order status tracking endpoint  
**So that** customers can monitor their orders

**Acceptance Criteria:**
- [ ] GET `/api/v1/orders/{order_id}/status` endpoint returns current order status
- [ ] Response includes order_id, status, and last_updated timestamp
- [ ] Status values are validated against allowed statuses: pending, in_progress, completed, cancelled
- [ ] Endpoint returns 404 if order_id does not exist
- [ ] Endpoint validates that authenticated user owns the order before returning status
- [ ] Returns 403 if user attempts to access another member's order
- [ ] Response includes estimated completion time for pending/in_progress orders (if available)

**Notes / Implementation Details:**
- Query MongoDB orders collection by order_id
- Add last_updated timestamp field to order documents
- Return minimal data for performance (just status-related fields)
- Consider WebSocket implementation for real-time status updates in future
- Log status check requests for analytics

**Estimated Effort:** Medium

---

### User Story E1-F5-US3: Order Details Retrieval Endpoint

**As a** developer  
**I want to** an endpoint to retrieve order details  
**So that** the web and chatbot experiences stay consistent

**Acceptance Criteria:**
- [ ] GET `/api/v1/orders/{order_id}` endpoint returns complete order details
- [ ] Response includes all order fields: order_id, location_id, order_date, items (with item details), total_amount, status, payment_method, notes
- [ ] Items array includes enriched data: item_id, quantity, unit_price, item_name, and category
- [ ] Endpoint validates that authenticated user owns the order
- [ ] Returns 404 if order does not exist
- [ ] Returns 403 if user is not authorized to view the order
- [ ] Location information is included (store_name, address) via location_id lookup

**Notes / Implementation Details:**
- Primary query to MongoDB for order document
- Secondary query to BigQuery to enrich item details (join menu_items table)
- Secondary query to BigQuery for location information (join locations table)
- Consider caching frequently accessed orders
- Implement data aggregation pipeline for performance
- Ensure consistent response format for both web and chatbot consumers

**Estimated Effort:** Large

---

### User Story E1-F5-US4: Order Filtering Capability

**As a** user  
**I want to** the ability to filter my orders  
**So that** I can see historical information about purchased items

**Acceptance Criteria:**
- [ ] Order history page includes filter controls for date range, status, and location
- [ ] Date range filter allows selecting start and end dates
- [ ] Status filter allows multi-select (pending, in_progress, completed, cancelled)
- [ ] Location filter shows user's frequented locations based on order history
- [ ] Filters can be combined (e.g., completed orders from Location A in last 30 days)
- [ ] Filter results update dynamically without full page reload
- [ ] Clear/reset filters button restores full order history view
- [ ] Active filters are displayed as tags/chips for easy removal

**Notes / Implementation Details:**
- Extend GET `/api/v1/orders/history/{member_id}` with query parameters:
  - `?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`
  - `?status=completed,cancelled`
  - `?location_id=LOC-001`
- Frontend implements filter UI with dropdowns and date pickers
- Build MongoDB aggregation pipeline for efficient filtering
- Consider implementing search by item name
- Store filter preferences in localStorage for user convenience

**Estimated Effort:** Large

---

## 
---

## Feature E1-F6: Infrastructure & Deployment
Establishes the containerized backend, CI/CD pipeline, environment separation, and GCP deployment for the API.

---

### User Story E1-F6-US1: CI/CD Pipeline Implementation

**As a** DevOps engineer  
**I want to** CI/CD pipelines  
**So that** new code deploys reliably

**Acceptance Criteria:**
- [ ] `.gitlab-ci.yml` file configured with stages: test, build, deploy
- [ ] Test stage runs all unit tests and fails pipeline if tests fail
- [ ] Build stage creates Docker container and pushes to Google Container Registry
- [ ] Deploy stage deploys container to Google Cloud Run
- [ ] Pipeline triggers automatically on commits to main/develop branches
- [ ] Deployment to production requires manual approval
- [ ] Pipeline status is visible in GitLab merge requests
- [ ] Failed deployments send notifications to team (email/Slack)

**Notes / Implementation Details:**
- Use GitLab CI/CD with GCP service account authentication
- Implement separate pipelines for API and website projects
- Include linting stage (pylint/flake8 for Python)
- Set up environment variables for GCP credentials in GitLab
- Use Cloud Run revision tags for easy rollbacks
- Implement deployment strategies: rolling updates or blue-green

**Estimated Effort:** Large

---

### User Story E1-F6-US2: Environment Separation

**As a** developer  
**I want to** environment separation (dev/stage/prod)  
**So that** releases are tested safely

**Acceptance Criteria:**
- [ ] Three separate GCP Cloud Run services configured: dev, staging, production
- [ ] Each environment has isolated database instances (BigQuery datasets, MongoDB databases)
- [ ] Environment-specific configuration files or environment variables (API keys, database URIs)
- [ ] Dev environment deploys automatically on commits to develop branch
- [ ] Staging environment deploys automatically on commits to staging branch
- [ ] Production environment requires manual approval before deployment
- [ ] Environment name is clearly indicated in API responses (e.g., in headers or logs)

**Notes / Implementation Details:**
- Use GitLab environment variables for environment-specific configs
- Naming convention: `uncle-joes-api-dev`, `uncle-joes-api-staging`, `uncle-joes-api-prod`
- Implement separate GCP projects or clearly labeled services within one project
- Use different BigQuery datasets: `uncle_joes_coffee_dev`, `uncle_joes_coffee_staging`, `uncle_joes_coffee_prod`
- Configure different MongoDB connection strings per environment
- Document environment URLs and access procedures

**Estimated Effort:** Medium

---

### User Story E1-F6-US3: API Containerization

**As a** DevOps engineer  
**I want to** containerized APIs  
**So that** deployments are consistent across environments

**Acceptance Criteria:**
- [ ] Dockerfile created for FastAPI application
- [ ] Docker image builds successfully and contains all dependencies
- [ ] Container runs FastAPI app on port 8080 (Cloud Run default)
- [ ] Docker image is optimized for size (multi-stage build, minimal base image)
- [ ] Health check endpoint (`/health`) responds correctly from container
- [ ] Container starts up in under 10 seconds
- [ ] Environment variables are passed to container at runtime (not hardcoded)

**Notes / Implementation Details:**
- Use official Python slim image as base (e.g., `python:3.11-slim`)
- Multi-stage build to reduce image size
- Copy only necessary files (use .dockerignore)
- Install dependencies via requirements.txt
- Use gunicorn or uvicorn as WSGI/ASGI server
- Set appropriate resource limits (CPU, memory) in Cloud Run
- Example CMD: `CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]`

**Estimated Effort:** Medium

---

### User Story E1-F6-US4: Logging and Tracing Implementation

**As a** developer  
**I want to** logging and tracing  
**So that** issues can be diagnosed quickly

**Acceptance Criteria:**
- [ ] Application logs are sent to Google Cloud Logging (Stackdriver)
- [ ] Logs include severity levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- [ ] Request/response logs include: timestamp, endpoint, status code, duration, user_id
- [ ] Error logs include stack traces and contextual information
- [ ] Logs are searchable and filterable in Cloud Logging console
- [ ] Distributed tracing implemented for multi-service requests
- [ ] Dashboard created for monitoring key metrics (request rate, error rate, latency)

**Notes / Implementation Details:**
- Use Python logging module with Cloud Logging handler
- Implement structured logging (JSON format) for better searchability
- Add correlation IDs to trace requests across services
- Use Google Cloud Trace for distributed tracing
- Log sensitive data exclusions (passwords, tokens)
- Set up log-based alerts for critical errors
- Configure log retention policies

**Estimated Effort:** Large

---

### User Story E1-F6-US5: API Response Schema Standardization

**As a** developer
**I want to** define a consistent schema for all API responses
**So that** the website and chatbot receive predictable formats

**Acceptance Criteria:**
- [ ] Standardized success, data, error envelope
- [ ] Consistent error codes across endpoints
- [ ] Schema documented in API reference
- [ ] Responses validated before returning

**Notes / Implementation Details:**
- Use Pydantic models in FastAPI
- Define global error handler
- Share schema across Website/Chatbot integrations

**Estimated Effort:** Medium

---

### User Story E1-F6-US6: Deploy Website via GCP Static Hosting (Vite)

**As a frontend** developer
**I want to** deploy the Vite-built site using Cloud Storage + CDN
**So that** the site loads quickly and meets GCP requirements

**Acceptance Criteria:**
- [ ] Vite build outputs static assets to /dist
- [ ] Assets uploaded to GCS bucket
- [ ] CDN configured via Cloud Load Balancer
- [ ] CI pipeline automates deployment

**Notes / Implementation Details:**
- Use gsutil rsync in CI
- Use GCP Load Balancer for HTTPS
- Cache-control headers for static hosting

**Estimated Effort:** Medium

---

User Story E1-F6-US7: Chatbot Deployment Pipeline

**As a** developer
**I want to** deploy the chatbot widget to GCP
**So that** it can be embedded across the website

**Acceptance Criteria:**
- [ ] Chatbot served through a Cloud Run service or CDN bucket
- [ ] Versioning handled automatically
- [ ] Embedded script loads reliably across pages
- [ ] CI pipeline builds and deploys Chatbot assets

**Notes / Implementation Details:**
- Deploy widget as static JavaScript bundle or iframe service
- Use semantic versioning for releases

**Estimated Effort:** Medium

---
## 
---

## Feature E1-F7: Testing & Quality Assurance
Ensures API reliability through unit tests, integration tests, load testing, and comprehensive staging validation.

---

### User Story E1-F7-US1: Acceptance Criteria Definition

**As a** PM  
**I want to** acceptance criteria defined  
**So that** features meet business expectations

**Acceptance Criteria:**
- [ ] All user stories have clear, testable acceptance criteria documented
- [ ] Acceptance criteria are reviewed and approved by stakeholders before development
- [ ] Criteria include both functional and non-functional requirements
- [ ] Edge cases and error scenarios are documented
- [ ] Definition of Done is established for each feature
- [ ] Acceptance criteria serve as basis for QA test cases
- [ ] Criteria are updated in project management tool (GitLab Issues) and linked to stories

**Notes / Implementation Details:**
- Use GIVEN-WHEN-THEN format for behavioral criteria where appropriate
- Include performance benchmarks (e.g., response time < 200ms)
- Specify security requirements (authentication, authorization)
- Document data validation rules
- Review criteria in sprint planning meetings
- Maintain traceability between requirements and test cases

**Estimated Effort:** Small

---

### User Story E1-F7-US2: Unit Test Creation for All Endpoints

**As a** QA engineer  
**I want to** create unit tests for all endpoints  
**So that** individual functions work correctly

**Acceptance Criteria:**
- [ ] Unit tests written for all API endpoints using pytest
- [ ] Test coverage is at least 80% for all endpoint functions
- [ ] Tests include positive test cases (valid inputs, expected outputs)
- [ ] Tests include negative test cases (invalid inputs, error handling)
- [ ] Tests use mocked database connections (no actual DB queries in unit tests)
- [ ] All tests pass in CI/CD pipeline before deployment
- [ ] Test results are reported in GitLab pipeline with pass/fail status

**Notes / Implementation Details:**
- Use pytest fixtures for test setup and teardown
- Mock BigQuery and MongoDB clients using pytest-mock or unittest.mock
- Organize tests in `tests/` directory mirroring app structure
- Test authentication/authorization logic separately
- Use pytest-cov for coverage reporting
- Include parametrized tests for multiple input scenarios
- Example: `tests/test_members_api.py`, `tests/test_orders_api.py`

**Estimated Effort:** Large

---

### User Story E1-F7-US3: Load Testing Implementation

**As a** QA engineer  
**I want to** load testing  
**So that** the API scales for peak hours

**Acceptance Criteria:**
- [ ] Load testing tool configured (e.g., Locust, JMeter, or k6)
- [ ] Load test scenarios simulate realistic traffic patterns (login, browse menu, view orders)
- [ ] Tests simulate peak load of 1000 concurrent users
- [ ] API maintains response time under 500ms at 90th percentile during load test
- [ ] No errors or timeouts occur under normal load conditions
- [ ] Load test results documented with graphs (response time, throughput, error rate)
- [ ] Bottlenecks identified and documented for optimization

**Notes / Implementation Details:**
- Use k6 or Locust for load testing (both have good cloud integration)
- Define realistic user scenarios with appropriate think time
- Test against staging environment initially
- Gradually increase load to identify breaking point
- Monitor Cloud Run metrics during tests (CPU, memory, instance count)
- Test autoscaling behavior
- Document findings and recommendations for optimization

**Estimated Effort:** Large

---

### User Story E1-F7-US4: Integration Test Implementation

**As a** QA engineer  
**I want to** implement integration tests  
**So that** API components work together properly

**Acceptance Criteria:**
- [ ] Integration tests verify end-to-end workflows (login → view orders → view order details)
- [ ] Tests use actual test database instances (separate from production)
- [ ] Tests verify database query results match expected data structure
- [ ] Tests validate API responses include correctly joined data (e.g., order items with menu details)
- [ ] Authentication flow is tested across multiple endpoints
- [ ] Integration tests run automatically in CI/CD pipeline (separate stage from unit tests)
- [ ] Test data is seeded before tests and cleaned up after

**Notes / Implementation Details:**
- Use pytest with actual database connections to test environments
- Create test fixtures for database seeding
- Implement teardown to clean test data after execution
- Test BigQuery and MongoDB integration separately and together
- Verify JWT token flow across protected endpoints
- Use Docker Compose for local integration testing environment
- Document integration test setup and execution procedures

**Estimated Effort:** Large

---

### User Story E1-F7-US5: Chatbot Integration Testing

**As a** QA engineer
**I want to** test chatbot interactions against API endpoints
**So that** all chatbot features remain functional

**Acceptance Criteria:**
- [ ] Tests validate location finder via API
- [ ] Tests validate order history lookups
- [ ] Tests handle invalid inputs for chatbot flows
- [ ] All API calls return correct and expected schema

**Notes / Implementation Details:**
- Use pytest + HTTPX for API tests
- Validate JSON schemas for all chatbot scenarios 
- Run in CI with coverage thresholds

Estimated Effort: Medium

---

### User Story E1-F7-US6: Staging Environment Setup

**As a** developer  
**I want to** a staging environment  
**So that** features are validated before release

**Acceptance Criteria:**
- [ ] Staging environment deployed on Google Cloud Run with production-like configuration
- [ ] Staging database (BigQuery dataset and MongoDB database) contains realistic test data
- [ ] Staging environment accessible via dedicated URL (e.g., `staging.unclejoes-api.com`)
- [ ] Staging deploys automatically when code is merged to staging branch
- [ ] Access to staging environment is restricted (authentication required or IP whitelisting)
- [ ] Staging environment mirrors production configuration (resource limits, environment variables)
- [ ] Testing team has documented access instructions and test accounts

**Notes / Implementation Details:**
- Create separate Cloud Run service for staging
- Use separate GCP project or clearly labeled staging resources
- Populate staging database with anonymized production data or synthetic test data
- Configure Basic Auth or IP whitelisting for staging access
- Set up staging-specific environment variables in GitLab
- Document staging URLs, credentials, and test data in team wiki
- Implement automatic deployment on merge to staging branch

**Estimated Effort:** Medium

---
## 
---

# Epic 2: Website
Customer-facing web application consuming the API.

---

## Feature E2-F1: Website Foundation
Core website structure, navigation, branding, and foundational UI components.

---

### User Story E2-F1-US1: Clean Navigation Menu

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

---

### User Story E2-F1-US2: Branded Homepage

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

---

### User Story E2-F1-US3: Quick Links to Menu and Locations

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

---

### User Story E2-F1-US4: Modern UI Template

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

---

### User Story E2-F1-US5: Error Boundary Handling

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
## 
---

## Feature E2-F2: Locations & Menu Display
Pages for browsing store locations and viewing menu items with search and filtering capabilities.

---

### User Story E2-F2-US1: Browse All Store Locations

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

---

### User Story E2-F2-US2: Retrieve List of Store Locations

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

---

### User Story E2-F2-US3: Search for Stores by Zip Code

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

---

### User Story E2-F2-US4: View Menu Visually

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

---

### User Story E2-F2-US5: View Detailed Product Information

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

---

### User Story E2-F2-US6: Filter Menu Items

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
##
---

## Feature E2-F3: Member Features
Member login, account dashboard, order history, and loyalty program features.

---

### User Story E2-F3-US1: View Loyalty Points and Rewards

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

---

### User Story E2-F3-US2: See Past Orders

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
- Handle empty state when user has no order history ("No orders yet. Start exploring our menu!")
- Ensure only authenticated user can view their own order history (JWT validation)

**Estimated Effort:** Large

---

### User Story E2-F3-US3: View Member Profile

**As a** member  
**I want to** view my profile  
**So that** I can see all of my information in one place

**Acceptance Criteria:**
- [ ] Profile page displays member_id, email, first_name, last_name, phone, and join_date
- [ ] Preferred location is shown with option to change
- [ ] Current loyalty points balance is displayed on profile page
- [ ] Account creation date (join_date) is formatted in readable format (e.g., "Member Since: January 2024")
- [ ] Profile page is accessible only when user is logged in
- [ ] Changes to profile auto-save or require explicit "Save Changes" action

**Notes / Implementation Details:**
- Fetch profile data from GET /api/v1/members/{member_id} endpoint using authenticated member_id
- Display fields in organized sections (Personal Info, Contact Info, Preferences)
- Show profile completion percentage if some fields are optional
- Display account security settings (email notifications, privacy preferences)
- Ensure password_hash is never displayed in frontend

**Estimated Effort:** Medium

---

### User Story E2-F3-US4: See Most Frequently Ordered Items

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

---

### User Story E2-F3-US5: User Login

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
---
###
---

# Epic 3: Chatbot
AI-powered chatbot for customer self-service.

---

## Feature E3-F1: Chatbot Widget & Foundation
Core UI and interaction layer for the chatbot experience.

---

### User Story E3-F1-US1: Embedded Chatbot Widget
**As a** visitor  
**I want to** have an embedded chatbot widget  
**So that** I can get help instantly  

**Acceptance Criteria:**  
- [ ] Chatbot widget loads on every page without breaking layout  
- [ ] Chat bubble opens and closes the chatbot panel smoothly  
- [ ] Widget remains fixed at bottom-right on all screen sizes  
- [ ] Widget does not interfere with main site navigation or clickable elements  

**Notes / Implementation Details:**  
- Implement floating widget container with CSS `position: fixed`  
- Use iframe or embedded React component for chatbot UI  
- Ensure z-index hierarchy avoids overlapping key UI elements  
- Consider lazy-loading chatbot script to reduce initial page load  

**Estimated Effort:** Large  

---

### User Story E3-F1-US2: Clear Options
**As a** user  
**I want to** see clear, simple options  
**So that** I know what the chatbot can help with  

**Acceptance Criteria:**  
- [ ] Quick actions appear when the chat window opens  
- [ ] Options include FAQs, Locations, Orders, and Help  
- [ ] Each quick action triggers a predefined intent  
- [ ] Options adapt to mobile format without breaking layout  

**Notes / Implementation Details:**  
- Store quick-action definitions in JSON file or config object  
- Render options dynamically inside chatbot UI  
- Use icons and short text labels for clarity  
- Consider LLM intent routing based on selected option  

**Estimated Effort:** Small  

---

### User Story E3-F1-US3: Mobile Support
**As a** customer  
**I want to** use the chatbot on mobile  
**So that** I can get help anywhere  

**Acceptance Criteria:**  
- [ ] Chatbot UI scales responsively under 480px screen width  
- [ ] Touch interactions open/close the widget smoothly  
- [ ] Chat window occupies full width on mobile when opened  
- [ ] No overlapping with mobile nav buttons  

**Notes / Implementation Details:**  
- Use responsive CSS breakpoints (`@media`)  
- Test on iOS Safari and Android Chrome  
- Ensure viewport meta tag is set in HTML  
- Implement smooth CSS transitions for mobile expansion  

**Estimated Effort:** Small  

---

### User Story E3-F1-US4: Chat Bubble Trigger
**As a** frontend developer  
**I want to** implement chat bubble trigger  
**So that** users can open/close chatbot  

**Acceptance Criteria:**  
- [ ] Floating chat bubble is visible on all pages  
- [ ] Clicking bubble toggles chatbot open/close  
- [ ] Bubble animates on hover for desktop  
- [ ] Bubble icon clearly indicates chatbot  

**Notes / Implementation Details:**  
- Use React component or vanilla JS event listener  
- Add accessibility attributes (`aria-expanded`)  
- Implement small animation (e.g., scale or bounce)  
- Asset: use SVG icon for crisp rendering  

**Estimated Effort:** Small  

---

### User Story E3-F1-US5: Integrate Chatbot SDK
**As a** developer  
**I want to** integrate chatbot SDK  
**So that** AI responses are available  

**Acceptance Criteria:**  
- [ ] SDK successfully initializes on page load  
- [ ] Chat messages are sent to SDK endpoint correctly  
- [ ] SDK responses appear in chatbot interface  
- [ ] Errors from SDK show fallback message  

**Notes / Implementation Details:**  
- Load SDK via npm or CDN  
- Ensure API keys stored in environment variables  
- Configure SDK session object for user tracking  
- Handle network failures with retries  

**Estimated Effort:** Medium  

---
##
---

## Feature E3-F2: FAQ Responses
Chatbot capability to answer common questions quickly.

---

### User Story E3-F2-US1: Answer FAQs
**As a** customer  
**I want to** get answers to FAQs  
**So that** I don't need to call support  

**Acceptance Criteria:**  
- [ ] Chatbot provides instant FAQ responses  
- [ ] FAQ responses match official documentation  
- [ ] FAQs trigger without requiring full-sentence queries  
- [ ] Fallback response provided if FAQ not found  

**Notes / Implementation Details:**  
- Store FAQs in JSON or CMS  
- Use keyword or intent-based matching  
- Preload FAQ dataset at chatbot initialization  
- Cache FAQs for performance  

**Estimated Effort:** Small  

---

### User Story E3-F2-US2: Natural Language Responses
**As a** customer  
**I want to** receive natural language responses  
**So that** interaction feels smooth  

**Acceptance Criteria:**  
- [ ] LLM-generated responses are clear and human-like  
- [ ] Responses remain within brand tone guidelines  
- [ ] Chatbot handles follow-up questions in the same thread  
- [ ] Fallback message displayed for ambiguous questions  

**Notes / Implementation Details:**  
- Use OpenAI or similar inference API  
- Provide conversation history context in each API call  
- Apply safety filters on model outputs  
- Consider few-shot prompting for consistency  

**Estimated Effort:** Small  

---

### User Story E3-F2-US3: External API for FAQs
**As a** developer  
**I want to** use external APIs  
**So that** chatbot can answer FAQs  

**Acceptance Criteria:**  
- [ ] External FAQ API responds within <500ms  
- [ ] Chatbot displays results correctly formatted  
- [ ] API authentication handled securely  
- [ ] Timeout fallback used when API fails  

**Notes / Implementation Details:**  
- Use Axios or Fetch for API calls  
- Implement retry logic for transient errors  
- Use API key stored server-side  
- Normalize API response into chatbot message format  

**Estimated Effort:** Medium  

---

### User Story E3-F2-US4: FAQ Dataset
**As a** developer  
**I want to** have FAQ document  
**So that** chatbot can be trained  

**Acceptance Criteria:**  
- [ ] Document contains at least 20–30 FAQs  
- [ ] FAQ file follows consistent schema  
- [ ] Chatbot loads dataset on startup  
- [ ] FAQ updates do not require redeployment  

**Notes / Implementation Details:**  
- Use JSON/YAML structured file  
- Host file in CDN or backend storage  
- Add versioning to FAQ dataset  
- Allow dynamic refresh via webhook or timed fetch  

**Estimated Effort:** Medium  

---
##
---

## Feature E3-F3: Order History Lookup
Enables chatbot to retrieve and display order-related information.

---

### User Story E3-F3-US1: Show Last Order
**As a** member  
**I want to** see my last order  
**So that** I can reference quickly  

**Acceptance Criteria:**  
- [ ] Chatbot retrieves last order from Orders API  
- [ ] Displays order items, date, and cost  
- [ ] Handles “no past orders found” gracefully  
- [ ] Requires user authentication  

**Notes / Implementation Details:**  
- Use GET `/orders/history/{member_id}?limit=1`  
- JWT required for API call  
- Format response as chatbot card UI  
- Cache result in session for quicker follow-ups  

**Estimated Effort:** Small  

---

### User Story E3-F3-US2: Ask About Specific Orders
**As a** member  
**I want to** ask about specific orders  
**So that** I get details  

**Acceptance Criteria:**  
- [ ] Chatbot asks for order ID or approximate date  
- [ ] Retrieves specific past order  
- [ ] Displays items, cost, and store location  
- [ ] Handles invalid order ID  

**Notes / Implementation Details:**  
- Query `/orders/{order_id}`  
- Use regex to extract order ID from user message  
- Provide clarifying questions if order is ambiguous  
- Display results in structured card format  

**Estimated Effort:** Small  

---

### User Story E3-F3-US3: API Consistency
**As a** developer  
**I want to** call Orders API  
**So that** chatbot uses real data  

**Acceptance Criteria:**  
- [ ] Responses match Orders API schema  
- [ ] All fields used are documented  
- [ ] Error handling covers 401, 403, 404  
- [ ] Timeout fallback functional  

**Notes / Implementation Details:**  
- Use HTTP client wrapper for repeat API calls  
- Create schema validation layer  
- Use exponential backoff for retries  
- Integrate with caching layer  

**Estimated Effort:** Small  

---

### User Story E3-F3-US4: Order Status Responses
**As a** developer  
**I want to** generate order status responses  
**So that** users can track orders  

**Acceptance Criteria:**  
- [ ] Chatbot retrieves current status  
- [ ] Shows friendly human-readable status  
- [ ] Handles cancelled or unknown orders  
- [ ] Provides next recommended steps  

**Notes / Implementation Details:**  
- Map API codes → text labels  
- Use `/orders/{order_id}/status` endpoint  
- Provide timestamp and location if available  
- Add escalation suggestion for issues  

**Estimated Effort:** Small  

---
##
---

## Feature E3-F4: Location Finder
Allows chatbot to search and provide location-related information.

---

### User Story E3-F4-US1: Find Nearby Stores
**As a** customer  
**I want to** find nearby stores  
**So that** I can visit one  

**Acceptance Criteria:**  
- [ ] Chatbot requests ZIP or city  
- [ ] Returns stores sorted by distance  
- [ ] Provides store name and address  
- [ ] Handles no-store-found scenario  

**Notes / Implementation Details:**  
- Use GET `/locations?zip=XXXX`  
- Use geolocation lookup API if permitted  
- Format store list as cards  
- Cache user ZIP for session  

**Estimated Effort:** Medium  

---

### User Story E3-F4-US2: Provide ZIP Code
**As a** customer  
**I want to** share ZIP  
**So that** results are accurate  

**Acceptance Criteria:**  
- [ ] ZIP input validated (5 digits)  
- [ ] ZIP is stored for repeated queries  
- [ ] Chatbot asks again if invalid ZIP  
- [ ] Graceful handling of missing ZIP  

**Notes / Implementation Details:**  
- Use regex validation  
- Store ZIP in chatbot context state  
- Integrate with location endpoint or map API  
- Add suggestions for common ZIP errors  

**Estimated Effort:** Small  

---

### User Story E3-F4-US3: Map Links
**As a** customer  
**I want to** get map links  
**So that** I can open directions quickly  

**Acceptance Criteria:**  
- [ ] Chatbot provides Google Maps links  
- [ ] Includes store name and coordinates  
- [ ] Link format works on mobile  
- [ ] Provides multiple map provider options (if available)  

**Notes / Implementation Details:**  
- Use standard URL formatting: `https://maps.google.com/?q=lat,long`  
- Pull lat/long from Locations API  
- Render link in clickable chat bubble  
- Consider deep linking for mobile apps  

**Estimated Effort:** Small  

---

### User Story E3-F4-US4: Store Hours Inquiry
**As a** customer  
**I want to** ask store hours  
**So that** I know when to visit  

**Acceptance Criteria:**  
- [ ] Chatbot retrieves store hours from API  
- [ ] Shows open/close status based on local time  
- [ ] Handles special hours or holidays  
- [ ] Provides fallback if data unavailable  

**Notes / Implementation Details:**  
- Query `/locations/{id}`  
- Convert store timezone to user timezone  
- Preprocess special hours if included  
- Detect if store is currently open  

**Estimated Effort:** Small  

---

### User Story E3-F4-US5: Integrate Location API
**As a** developer  
**I want to** integrate Location API  
**So that** chatbot can access store data  

**Acceptance Criteria:**  
- [ ] Successful API integration  
- [ ] Handles 404 store not found  
- [ ] Returns consistent data format  
- [ ] Handles network errors  

**Notes / Implementation Details:**  
- Use centralized fetch wrapper  
- Define interface for location response  
- Add caching for repeated calls  
- Implement error logging  

**Estimated Effort:** Medium  

---
##
---

## Feature E3-F5: Chatbot-Website Integration
Ensures seamless behavior between the chatbot and the website UI.

---

### User Story E3-F5-US1: Open Website Suggestions
**As a** visitor  
**I want to** open chatbot suggestions in website  
**So that** I can keep browsing  

**Acceptance Criteria:**  
- [ ] Chatbot sends clickable suggestions  
- [ ] Links open correct website pages  
- [ ] Formatting consistent across devices  
- [ ] No interference with website navigation  

**Notes / Implementation Details:**  
- Use deep-link structure `/menu`, `/locations`, etc.  
- Send suggestions using structured chatbot messages  
- Validate URL before rendering  
- Add analytics tracking for clicks  

**Estimated Effort:** Medium  

---

### User Story E3-F5-US2: Chatbot Analytics
**As an** admin  
**I want to** see analytics  
**So that** I understand user behavior  

**Acceptance Criteria:**  
- [ ] Track chatbot session count  
- [ ] Track feature usage per session  
- [ ] Track failed queries  
- [ ] Export insights via dashboard  

**Notes / Implementation Details:**  
- Use Google Analytics or Mixpanel  
- Add event emitter to chatbot actions  
- Batch events for performance  
- Store anonymized data only  

**Estimated Effort:** Medium  

---

### User Story E3-F5-US3: Clean Integration
**As a** developer  
**I want to** ensure clean integration  
**So that** systems feel unified  

**Acceptance Criteria:**  
- [ ] Consistent styling with site theme  
- [ ] Shared fonts, colors, and behavior  
- [ ] No layout shifts  
- [ ] No console errors related to integration  

**Notes / Implementation Details:**  
- Share CSS theme variables  
- Use React component library if frontend supports  
- Validate integration across browsers  
- Run integration tests via Cypress  

**Estimated Effort:** Medium  

---

### User Story E3-F5-US4: Persist Chat History
**As a** developer  
**I want to** persist chat history in session  
**So that** chatbot maintains context  

**Acceptance Criteria:**  
- [ ] Chat persists between page reloads  
- [ ] History cleared after session timeout  
- [ ] Only user's messages stored  
- [ ] Respects privacy rules  

**Notes / Implementation Details:**  
- Use sessionStorage  
- Store last 20 messages max  
- Clear on logout or session end  
- Avoid storing PII  

**Estimated Effort:** Medium  

---

### User Story E3-F5-US5: Human Handoff
**As a** developer  
**I want to** implement human handoff  
**So that** complex issues escalate  

**Acceptance Criteria:**  
- [ ] Chatbot detects escalation intent  
- [ ] Forwards conversation summary  
- [ ] Provides contact options  
- [ ] Logs escalation events  

**Notes / Implementation Details:**  
- Implement keyword detection (“agent”, “help”, etc.)  
- Summarize conversation via LLM  
- Provide email/phone/live chat links  
- Add routing rules for escalation events  

**Estimated Effort:** Medium  