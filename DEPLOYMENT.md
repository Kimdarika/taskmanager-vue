# Deployment Guide

This project only needs these folders for deployment:

- `backend/`
- `frontend/`

The `backend_scaffold_backup/` folder is only a backup copy and is not used by the app.

## Backend

1. Configure MySQL in `backend/.env`.
2. Run migrations:

```bash
cd backend
php artisan migrate --force
```

3. Seed sample data if needed:

```bash
php artisan db:seed --force
```

4. Start the API:

```bash
php artisan serve
```

## Frontend

1. Install dependencies:

```bash
cd frontend
npm install
```

2. Build for production:

```bash
npm run build
```

## Notes

- The frontend talks to the backend through `/api`.
- Make sure the backend URL and CORS settings match your hosting setup.
- If you remove `backend_scaffold_backup/`, the app itself will still work because it is not referenced by the code.
