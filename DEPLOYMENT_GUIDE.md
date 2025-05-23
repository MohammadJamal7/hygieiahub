# Deployment Guide for Fixing 504 Gateway Timeout Issues

This guide provides step-by-step instructions to fix the 504 Gateway Timeout issues on your Amezmo-hosted Laravel application.

## 1. Update Environment Configuration

Replace your current `.env` file with the optimized `.env.production` file:

```bash
# On your Amezmo server
cp .env.production .env
php artisan key:generate
```

## 2. Apply PHP Configuration

Upload the `php.ini` file to your Amezmo server and configure it to be used:

1. Log in to your Amezmo dashboard
2. Go to Settings > PHP
3. Upload the `php.ini` file or copy its contents to the PHP configuration section
4. Save the changes

## 3. Install Required Packages

Install the Intervention/Image package for image optimization:

```bash
chmod +x install_image_package.sh
./install_image_package.sh
```

## 4. Run Database Optimizations

Run the migration to add indexes to your database tables:

```bash
php artisan migrate
```

If you encounter any issues with the first migration, you can try the alternative migration:

```bash
# Run only the alternative migration
php artisan migrate --path=database/migrations/2025_05_13_000001_add_indexes_alternative.php
```

If you still encounter issues, you can manually add the indexes using MySQL commands. Connect to your database and run:

```sql
-- For questionnaire_ones table
ALTER TABLE questionnaire_ones ADD INDEX questionnaire_ones_user_id_index (user_id);
ALTER TABLE questionnaire_ones ADD INDEX questionnaire_ones_supervisor_id_index (supervisor_id);
ALTER TABLE questionnaire_ones ADD INDEX questionnaire_ones_task_date_index (task_date);
ALTER TABLE questionnaire_ones ADD INDEX questionnaire_ones_status_index (status);

-- For questionnaire2s table
ALTER TABLE questionnaire2s ADD INDEX questionnaire2s_user_id_index (user_id);
ALTER TABLE questionnaire2s ADD INDEX questionnaire2s_task_date_index (task_date);
ALTER TABLE questionnaire2s ADD INDEX questionnaire2s_status_index (status);

-- For users table
ALTER TABLE users ADD INDEX users_role_index (role);
```

Note: When running the SQL commands directly, you might get errors if the indexes already exist. You can ignore these errors.

## 5. Configure Nginx (via Amezmo Dashboard)

Add the following to your Nginx configuration in the Amezmo dashboard:

```nginx
# Increase timeouts
fastcgi_read_timeout 120s;
fastcgi_connect_timeout 120s;
fastcgi_send_timeout 120s;
proxy_read_timeout 120s;
proxy_connect_timeout 120s;
proxy_send_timeout 120s;
client_body_timeout 120s;
client_header_timeout 120s;
```

## 6. Restart Services

Restart PHP-FPM and Nginx services:

```bash
# Through Amezmo dashboard or CLI if available
sudo service php-fpm restart
sudo service nginx restart
```

## 7. Monitor Performance

After implementing these changes, monitor your application's performance:

1. Check the logs for any remaining timeout issues
2. Monitor server resource usage (CPU, memory)
3. Test the application under load to ensure stability

## Troubleshooting

If you still experience timeout issues:

1. Check the Laravel logs: `storage/logs/laravel.log`
2. Check Nginx error logs
3. If you encounter migration errors:
   - You can try running `php artisan migrate:fresh` (WARNING: This will delete all data)
   - Or manually add indexes using MySQL commands
4. Consider further optimizations:
   - Implement queue processing for heavy tasks
   - Optimize database queries further
   - Consider using Redis for caching instead of file/database

## Additional Recommendations

1. **Image Optimization**: Consider using a CDN for serving images
2. **Database Maintenance**: Regularly run `OPTIMIZE TABLE` on your MySQL tables
3. **Caching**: Implement more aggressive caching for static content
4. **Monitoring**: Set up monitoring tools to alert you of performance issues
