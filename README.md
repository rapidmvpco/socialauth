This is the Flutterflow function for adding into your custom action. You will need to rename the function and change the `provider.github` to whichever social provider you are using.

```
import 'package:supabase/supabase.dart';

Future<void> githubAuth() async {
  try {
    await SupaFlow.client.auth.signInWithOAuth(Provider.github);
  } catch (e) {
    print('Exception: $e');
  }
}

'''

To create a trigger to populate the public.users table this is the code for the SQL editor

```
-- Create the trigger
CREATE TRIGGER after_auth_users_insert
AFTER INSERT ON auth.users
FOR EACH ROW
EXECUTE FUNCTION updatepublicusers();

```

To remove the trigger from your database use this

```
DROP TRIGGER after_auth_users_insert ON auth.users

```

The actual function you are calling with the trigger is this one. Remeber you will need to add a search_path as this is a security definer function. This only populates the email and userid. You will need to add further fields if required.

```
CREATE OR REPLACE FUNCTION updatepublicusers()
RETURNS TRIGGER SECURITY definer AS $$
BEGIN
    -- Check if the operation is an INSERT
    IF TG_OP = 'INSERT' THEN
        -- Insert or update the corresponding record in public.users
        INSERT INTO public.users (userid, email)
        VALUES (NEW.id, NEW.email)
        ON CONFLICT (userid) DO UPDATE
        SET email = EXCLUDED.email;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```
