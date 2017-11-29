The profiles API is based around two core interfaces.

### `Profile`
A profile represents the unique id + (optional) username of a player on the server.

```java
Player p = ...;
Profile profile = Profile.create(p.getUniqueId(), p.getName());
```

### `ProfileRepository`
Profile repositories contain a cache of known profiles, and provide methods to lookup profiles by unique id or username.

A base implementation of this interface is provided by the `helper-profiles` plugin.

#### Getting a profile by name
```java
ProfileRepository repo = getService(ProfileRepository.class);

Optional<Profile> lucksProfile = repo.getProfile("Luck");
if (lucksProfile.isPresent()) {
    UUID lucksUniqueId = lucksProfile.get().getUniqueId(); // wew!
}
```

#### Getting a profile by uuid
```java
Profile lucksProfile = repo.getProfile(UUID.fromString("c1d60c50-70b5-4722-8057-87767557e50d"));
Optional<String> lucksName = lucksProfile.getName();
```

#### Looking up a profile
The #get methods will only query the local in-memory cache for profiles, whereas the #lookup methods will query the backing database (and therefore return promises)
```java
repo.lookupProfile("Notch")
        .thenAcceptSync(profile -> {
            if (profile.isPresent()) {
                UUID uniqueId = profile.get().getUniqueId();
            }
        });
```

#### Looking up multiple profiles
When getting multiple profiles, it's better to use the combined methods, as the request can be combined into a single SQL query.
```java
Promise<Map<String, Profile>> profiles = repo.lookupProfilesByName(Arrays.asList("Luck", "lucko", "Notch"));

profiles.thenAcceptAsync(result -> {
    Profile luck = result.get("Luck");
    Profile lucko = result.get("lucko");
    Profile helper = result.get("Notch");

    // Do something
});
```