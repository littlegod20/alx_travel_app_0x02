# ALX Travel App 0x02 - Chapa Payment Integration

A Django-based travel booking platform with integrated Chapa payment gateway for secure payment processing.

## Features

- **Travel Listings Management**: Create and manage travel accommodation listings
- **Booking System**: Users can create bookings for listings
- **Chapa Payment Integration**: Secure payment processing via Chapa API
- **Payment Verification**: Real-time payment status verification
- **Email Confirmations**: Automated booking confirmation emails via Celery
- **RESTful API**: Complete REST API for all operations

## Chapa Payment Integration

This project integrates the Chapa payment gateway to handle secure payment processing for bookings. When a user creates a booking, the system automatically initiates a payment process and provides a checkout URL for completing the payment.

### Payment Workflow

1. **Booking Creation**: User creates a booking through the API
2. **Payment Initiation**: System automatically creates a Payment record and initiates payment with Chapa
3. **Payment Completion**: User completes payment via Chapa checkout URL
4. **Payment Verification**: System verifies payment status with Chapa API
5. **Booking Confirmation**: On successful payment, booking status is updated to "confirmed"
6. **Email Notification**: Confirmation email is sent to the guest via Celery background task

## Setup Instructions

### Prerequisites

- Python 3.8+
- MySQL database
- RabbitMQ (for Celery)
- Chapa API account (https://developer.chapa.co/)

### Installation

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd alx_travel_app_0x02
   ```

2. **Create and activate virtual environment**:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r alx_travel_app/requirement.txt
   ```

4. **Configure environment variables**:
   Create a `.env` file in the project root with the following variables:
   ```env
   SECRET_KEY=your-secret-key
   DEBUG=True
   DB_NAME=your_database_name
   DB_USER=your_database_user
   DB_PASSWORD=your_database_password
   DB_HOST=localhost
   DB_PORT=3306
   CHAPA_SECRET_KEY=your-chapa-secret-key
   EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
   EMAIL_HOST=smtp.gmail.com
   EMAIL_PORT=587
   EMAIL_USE_TLS=True
   EMAIL_HOST_USER=your-email@gmail.com
   EMAIL_HOST_PASSWORD=your-email-password
   DEFAULT_FROM_EMAIL=noreply@alxtravelapp.com
   ```

5. **Set up Chapa API**:
   - Create an account at https://developer.chapa.co/
   - Obtain your API secret key from the dashboard
   - Add the secret key to your `.env` file as `CHAPA_SECRET_KEY`
   - For testing, use Chapa's sandbox environment

6. **Run database migrations**:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

7. **Create superuser** (optional):
   ```bash
   python manage.py createsuperuser
   ```

8. **Start RabbitMQ** (for Celery):
   ```bash
   # On Linux/Mac
   sudo systemctl start rabbitmq-server
   
   # On Windows, install RabbitMQ and start the service
   ```

9. **Start Celery worker** (in a separate terminal):
   ```bash
   celery -A alx_travel_app.listings.celery worker --loglevel=info
   ```

10. **Start the development server**:
    ```bash
    python manage.py runserver
    ```

## API Endpoints

### Listings
- `GET /api/listings/` - List all listings
- `GET /api/listings/{id}/` - Get a specific listing
- `POST /api/listings/` - Create a new listing
- `PUT /api/listings/{id}/` - Update a listing
- `DELETE /api/listings/{id}/` - Delete a listing

### Bookings
- `GET /api/bookings/` - List all bookings
- `GET /api/bookings/{id}/` - Get a specific booking
- `POST /api/bookings/` - Create a new booking (automatically initiates payment)
- `PUT /api/bookings/{id}/` - Update a booking
- `DELETE /api/bookings/{id}/` - Delete a booking

### Payments
- `GET /api/payments/` - List all payments
- `GET /api/payments/{id}/` - Get a specific payment
- `POST /api/payments/` - Create and initiate a payment
- `GET /api/payments/{id}/verify/` - Verify payment status
- `POST /api/payments/{id}/verify/` - Verify payment status

## Testing Chapa Integration

### Using Chapa Sandbox

1. **Get Test Credentials**:
   - Log in to Chapa dashboard
   - Navigate to API settings
   - Use the sandbox/test API key for testing

2. **Test Payment Flow**:
   ```bash
   # 1. Create a booking
   curl -X POST http://localhost:8000/api/bookings/ \
     -H "Content-Type: application/json" \
     -d '{
       "listing_id": 1,
       "guest_id": 1,
       "check_in_date": "2024-12-01",
       "check_out_date": "2024-12-05",
       "number_of_guests": 2,
       "email": "test@example.com",
       "phone": "+251911234567"
     }'
   
   # Response will include payment checkout_url
   
   # 2. Complete payment on Chapa checkout page
   # Use test card: 4242 4242 4242 4242
   
   # 3. Verify payment
   curl -X GET http://localhost:8000/api/payments/{payment_id}/verify/
   ```

3. **Test Cards** (Chapa Sandbox):
   - Card Number: `4242 4242 4242 4242`
   - Expiry: Any future date
   - CVV: Any 3 digits
   - PIN: Any 4 digits

### Expected Behavior

- **Booking Creation**: Returns booking with payment checkout URL
- **Payment Initiation**: Payment status set to "pending"
- **Payment Verification**: After successful payment, status updates to "completed"
- **Booking Confirmation**: Booking status changes to "confirmed"
- **Email Confirmation**: Email sent to guest (check Celery logs)

## Project Structure

```
alx_travel_app_0x02/
├── alx_travel_app/
│   ├── listings/
│   │   ├── models.py          # Payment, Booking, Listing models
│   │   ├── views.py           # PaymentViewSet, BookingViewSet
│   │   ├── serializers.py     # PaymentSerializer
│   │   ├── chapa_service.py   # Chapa API integration
│   │   ├── tasks.py          # Celery tasks for emails
│   │   ├── urls.py           # API routes
│   │   └── admin.py          # Admin configuration
│   ├── settings.py           # Django settings with Chapa config
│   └── requirement.txt       # Python dependencies
├── manage.py
└── README.md
```

## Payment Model

The Payment model stores:
- `booking`: Foreign key to Booking
- `transaction_id`: Chapa transaction reference
- `amount`: Payment amount
- `status`: Payment status (pending, completed, failed)
- `payment_reference`: Unique payment reference
- `chapa_response`: Full Chapa API response (JSON)

## Error Handling

The integration includes comprehensive error handling for:
- Chapa API failures (network errors, API errors)
- Payment verification failures
- Email sending failures
- Invalid payment data

All errors are logged and appropriate status codes are returned to API consumers.

## Security Notes

- **API Keys**: Never commit `CHAPA_SECRET_KEY` to version control
- **Environment Variables**: Store all sensitive data in `.env` file
- **HTTPS**: Use HTTPS in production for all API calls
- **Validation**: All payment amounts and references are validated
- **Authentication**: Implement proper authentication/authorization for production

## Troubleshooting

### Payment Not Initiating
- Check `CHAPA_SECRET_KEY` is set correctly
- Verify Chapa API is accessible
- Check network connectivity
- Review application logs

### Payment Verification Failing
- Ensure transaction_id is correct
- Check Chapa API response
- Verify payment was completed on Chapa side
- Review verification logs

### Email Not Sending
- Verify Celery worker is running
- Check email configuration in settings
- Review Celery task logs
- Ensure email backend is configured correctly

## Documentation

- **Chapa API Docs**: https://developer.chapa.co/
- **Django REST Framework**: https://www.django-rest-framework.org/
- **Celery Documentation**: https://docs.celeryproject.org/

## License

This project is part of the ALX ProDev Backend curriculum.

## Author

ALX ProDev Student
