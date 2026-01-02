# Build Variants Configuration

This service now supports multiple build variants: **development**, **preview**, and **release**.

## Usage

### CLI

```bash
# Development build
build-service build --variant development

# Preview build
build-service build --variant preview

# Release build (default)
build-service build --variant release
build-service build  # Same as above
```

### What Each Variant Does

**Development**
- Debug mode enabled
- Source maps included
- No code obfuscation
- Faster build time
- Gradle command: `assembleDebug` or `assembleDevelopment`

**Preview**
- Optimized but debuggable
- Good for testing before release
- Can include staging API endpoints
- Gradle command: `assemblePreview`

**Release**
- Fully optimized
- Code minified and obfuscated
- Production ready
- Smallest APK size
- Gradle command: `assembleRelease`

## Setting Up Variants in Your Project

### Option 1: Use Expo Build Profiles

Add to your `app.json`:

```json
{
  "expo": {
    "android": {
      "buildTypes": {
        "development": {},
        "preview": {},
        "release": {}
      }
    }
  }
}
```

### Option 2: Configure Gradle Build Variants

After running `expo prebuild`, edit `android/app/build.gradle`:

```gradle
android {
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
            versionNameSuffix "-debug"
            debuggable true
        }
        
        development {
            applicationIdSuffix ".dev"
            versionNameSuffix "-dev"
            debuggable true
            minifyEnabled false
        }
        
        preview {
            applicationIdSuffix ".preview"
            versionNameSuffix "-preview"
            debuggable true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### Option 3: Product Flavors (Advanced)

For multiple app variants with different features:

```gradle
android {
    flavorDimensions "version"
    
    productFlavors {
        free {
            dimension "version"
            applicationIdSuffix ".free"
        }
        
        paid {
            dimension "version"
            applicationIdSuffix ".paid"
        }
    }
}
```

Then build with:
```bash
build-service build --variant free
build-service build --variant paid
```

## Environment Variables Per Variant

Create variant-specific environment files:

```
.env.development
.env.preview  
.env.production
```

Update the workflow to load the correct env file:

```yaml
- name: Create .env file
  run: |
    VARIANT="${{ steps.build_info.outputs.VARIANT }}"
    if [ -f "project/.env.$VARIANT" ]; then
      cp project/.env.$VARIANT project/.env
    fi
```

## Examples

### Development Build
```bash
build-service build --variant development
```
**Output:** `build-1767264765206-1767264800-development.apk`

### Preview Build
```bash
build-service build --variant preview
```
**Output:** `build-1767264765206-1767264800-preview.apk`

### Release Build
```bash
build-service build --variant release
# or simply
build-service build
```
**Output:** `build-1767264765206-1767264800-release.apk`

## Build Matrix (Multiple Variants at Once)

Want to build all variants in one go? Update workflow:

```yaml
strategy:
  matrix:
    variant: [development, preview, release]

steps:
  - name: Build ${{ matrix.variant }}
    run: ./gradlew assemble${{ matrix.variant }} --no-daemon
```

Then trigger with all variants in the payload.

## Artifact Naming

Artifacts are now named with the variant:
- `build-{buildId}-{timestamp}-development.apk`
- `build-{buildId}-{timestamp}-preview.apk`
- `build-{buildId}-{timestamp}-release.apk`

This makes it easy to identify which build is which!
