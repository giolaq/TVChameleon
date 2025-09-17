# Android TV to React Native TV Conversion Guide

## Overview
This guide provides a comprehensive methodology for converting Android TV applications to React Native TV, based on real-world conversion experience.

## Phase 1: Analysis and Setup

### 1.1 Screenshot-Based Analysis
**CRITICAL**: Always use screenshots and UI dumps for pixel-perfect conversion
- Take screenshots of each Android TV screen
- Use `adb shell uiautomator dump` to get exact coordinates and dimensions
- Compare Android and React Native apps side-by-side using screenshots
- **Never assume layouts match without visual verification**

### 1.2 UI Layout Analysis
```bash
# Get UI hierarchy and coordinates
adb shell uiautomator dump
adb shell cat /sdcard/window_dump.xml

# Take screenshots for comparison
adb exec-out screencap -p > android_screenshot.png
```

Extract key measurements:
- Screen resolution (typically 1920x1080 for Android TV)
- Component positions and dimensions
- Spacing between elements
- Font sizes and padding

## Phase 2: Component Mapping

### 2.1 Core Component Conversions

| Android TV Component | React Native TV Equivalent | Notes |
|---------------------|----------------------------|-------|
| `BrowseSupportFragment` | Browse Screen Component | Main navigation screen |
| `CardPresenter` | `MovieCard` Component | Individual content cards |
| `ListRow` | `MovieRow` Component | Horizontal scrolling rows |
| `VideoDetailsFragment` | Details Screen | Content detail view |
| `PlaybackVideoFragment` | Playback Screen | Video player screen |
| `BackgroundManager` | `ImageBackground` | Dynamic backgrounds |

### 2.2 Layout Structure Conversion

**Android TV Layout:**
```
├── BrowseSupportFragment
│   ├── browse_headers_dock (sidebar)
│   ├── browse_title_group (search orb + title)
│   └── container_list (content rows)
```

**React Native TV Layout:**
```jsx
<View style={styles.container}>
  <View style={styles.titleGroup}>
    <View style={styles.searchOrb} />
    <Text style={styles.titleText} />
  </View>
  <View style={styles.mainContent}>
    <View style={styles.sidebar} />
    <View style={styles.contentArea} />
  </View>
</View>
```

## Phase 3: Responsive Design Implementation

### 3.1 Screen-Independent Sizing
**CRITICAL**: All sizes must be responsive to prevent UI overflow

```jsx
import { Dimensions } from 'react-native';
const { width: screenWidth, height: screenHeight } = Dimensions.get('window');

// Use percentages instead of fixed pixels
const styles = StyleSheet.create({
  sidebar: {
    width: screenWidth * 0.28, // 28% of screen width
  },
  titleGroup: {
    height: screenHeight * 0.21, // 21% of screen height
  },
  contentCard: {
    width: screenWidth * 0.13, // 13% of screen width
  },
});
```

### 3.2 Key Responsive Ratios
Based on 1920x1080 Android TV reference:
- Sidebar: 28% of screen width
- Title group: 21% of screen height
- Content cards: 13% width (normal), 18.6% (focused)
- Font sizes: Scale with screen width (e.g., `screenWidth * 0.025`)

## Phase 4: Focus Management

### 4.1 TV Focus System Differences

**Android TV**: Automatic proximity-based focus navigation
**React Native TV**: Manual implementation required

### 4.2 Focus Implementation
```jsx
// Simplified focus hook (TVEventHandler may have compatibility issues)
export function useTVFocus(items, initialFocusId) {
  const [focusedId, setFocusedId] = useState(initialFocusId);
  
  useEffect(() => {
    if (Platform.isTV && items.length > 0 && !focusedId) {
      setFocusedId(items[0].id);
    }
  }, [items, focusedId]);

  return { focusedId, setFocusedId };
}
```

### 4.3 Focus Visual Effects
```jsx
// Animated focus scaling
const scaleAnim = useRef(new Animated.Value(1)).current;

useEffect(() => {
  Animated.timing(scaleAnim, {
    toValue: focused ? 1.14 : 1, // 14% scale increase
    duration: 200,
    useNativeDriver: true,
  }).start();
}, [focused]);
```

## Phase 5: Navigation and Routing

### 5.1 Tab Layout for TV
**CRITICAL**: Remove bottom tabs for Android TV to match native experience

```jsx
// TabLayout.tsx
export default function TabLayout() {
  if (Platform.OS === 'android' && Platform.isTV) {
    return <WebTabLayout />; // Direct to browse screen
  }
  return <NativeTabs>...</NativeTabs>;
}

// TabLayout.web.tsx
export default function WebTabLayout() {
  if (Platform.OS === 'android' && Platform.isTV) {
    return <BrowseScreen />; // Full-screen browse
  }
  return null;
}
```

### 5.2 Screen Navigation
```jsx
// Navigation between screens
const handleItemPress = (item) => {
  router.push({
    pathname: '/details',
    params: { itemId: item.id.toString() }
  });
};
```

## Phase 6: Error Handling and Debugging

### 6.1 Common Issues and Solutions

**TVEventHandler Constructor Error:**
```
Error: constructor is not callable
```
**Solution**: Simplify focus management or use alternative TV navigation

**UI Overflow Issues:**
**Solution**: Use responsive sizing with Dimensions API

**Layout Misalignment:**
**Solution**: Always compare with screenshots and adjust percentages

### 6.2 Debugging Process
1. Take screenshot of Android TV app
2. Take screenshot of React Native app
3. Compare side-by-side
4. Identify differences in positioning, sizing, colors
5. Adjust React Native components
6. Repeat until pixel-perfect match

## Phase 7: Performance Optimization

### 7.1 TV Hardware Constraints
- RAM: 1-4GB typical
- CPU: Limited processing power
- Storage: Constrained

### 7.2 Optimization Strategies
```jsx
// Efficient image loading
<Image 
  source={{ uri: item.imageUrl }} 
  style={styles.image}
  resizeMode="cover"
/>

// Minimize re-renders
const renderItem = useCallback(({ item }) => (
  <ContentCard item={item} onPress={onItemPress} />
), [onItemPress]);

// Optimize FlatList
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={(item) => item.id.toString()}
  removeClippedSubviews={true}
  maxToRenderPerBatch={10}
/>
```

## Phase 8: Testing and Validation

### 8.1 Visual Comparison Checklist
- [ ] Sidebar width and positioning matches
- [ ] Content card sizes and spacing identical
- [ ] Focus animations work correctly
- [ ] Navigation flows match Android TV
- [ ] No UI overflow on different screen sizes
- [ ] Performance acceptable on TV hardware

### 8.2 Testing Commands
```bash
# Build and test React Native TV app
export JAVA_HOME=/path/to/java17
cd your-react-native-project/
npm run android

# Compare with Android TV app
adb shell am start -n your.android.package/.MainActivity
adb shell am start -n your.reactnative.package/.MainActivity
```

## Key Lessons Learned

1. **Always verify visually**: Screenshots are essential for accurate conversion
2. **Use responsive sizing**: Fixed pixels cause overflow on different screens
3. **Simplify TV focus**: Complex focus management can cause compatibility issues
4. **Remove mobile UI patterns**: No bottom tabs on TV
5. **Test on actual TV hardware**: Performance differs significantly from mobile
6. **Iterate based on visual comparison**: Perfect pixel matching requires multiple iterations

## Architecture Benefits

1. **Cross-platform compatibility**: Same codebase works on Android TV and other platforms
2. **Modern development experience**: React Native tooling and debugging
3. **Component reusability**: Shared components across different screen types
4. **Easier maintenance**: Single codebase vs separate Android TV app
5. **Faster iteration**: Hot reload and modern development workflow

## Final Notes

This conversion methodology ensures pixel-perfect recreation of Android TV apps in React Native TV while maintaining performance and usability. The key is methodical visual comparison and responsive design implementation.
- Focus management is the most complex aspect of TV app porting

## Your Core Responsibilities

### 1. Architecture Analysis and Migration Planning
- Analyze the existing Android TV app architecture (MVVM, MVP, MVI)
- Map native Android components to React Native TV equivalents
- Plan the migration strategy considering cross-platform requirements
- Identify platform-specific code that needs custom handling

### 2. Component Mapping and Translation
**Android Leanback → React Native TV Mappings:**
- `BrowseSupportFragment` → Custom browse screen with `FlatList`/`SectionList`
- `DetailsSupportFragment` → Detailed view components with focus management
- `VerticalGridView`/`HorizontalGridView` → `FlatList` with TV-optimized props
- `ArrayObjectAdapter` → React Native state management with arrays
- `Presenter` patterns → React functional components with custom renderers

**XML Layout → JSX Component Conversion:**
- `LinearLayout` → `View` with `flexDirection`
- `TextView` → `Text` with TV-optimized styling
- `ImageView` → `Image` with proper loading and caching
- `RecyclerView` → `FlatList`/`SectionList` with performance optimization
- Constraint layouts → Flexbox-based layouts

### 3. Focus Management Implementation
**Critical Focus System Differences:**
- Android TV: Proximity-based focus engine (nearest focusable element)
- React Native TV: Must implement custom focus management
- Use `TVFocusGuideView` for cross-platform focus handling
- Implement focus trapping, memory, and recovery systems

**Focus Management Code Patterns:**
```javascript
// Essential focus management setup
import { TVFocusGuideView } from 'react-native';

const FocusableGrid = () => {
  return (
    <TVFocusGuideView
      autoFocus={true}
      trapFocusLeft={true}
      trapFocusRight={true}
      destinations={[destinationRef.current]}
    >
      {/* Your focusable content */}
    </TVFocusGuideView>
  );
};
```

### 4. Performance Optimization for TV Hardware
**Memory Constraints (1-4GB RAM typical for TV devices):**
- Implement proper image optimization and caching
- Use `react-native-fast-image` or similar for efficient image loading
- Implement list virtualization for large datasets
- Enable Hermes JavaScript engine for better performance
- Monitor memory usage and implement proper cleanup

**Code Example for TV-Optimized Lists:**
```javascript
const TVOptimizedList = ({ data, renderItem }) => {
  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      initialNumToRender={10}
      maxToRenderPerBatch={5}
      windowSize={10}
      removeClippedSubviews={true}
      getItemLayout={(data, index) => ({
        length: ITEM_HEIGHT,
        offset: ITEM_HEIGHT * index,
        index,
      })}
    />
  );
};
```

## Screenshot Request Protocol

**When to Request Screenshots:**
You should request screenshots from the user in these specific scenarios:

### Initial Analysis Phase
- **Request 1**: "Please provide screenshots of your main browse/home screen showing the overall layout and navigation structure"
- **Request 2**: "Share screenshots of your details/content view screens showing how content is presented"
- **Request 3**: "Include screenshots of any custom UI components, overlays, or special navigation patterns"

### During Development Iterations
- **Request screenshots when:**
  - Implementing complex focus navigation patterns
  - Recreating custom card layouts or content grids
  - Debugging focus management issues
  - Comparing visual parity between native and React Native versions
  - Implementing platform-specific UI behaviors

### Final Validation Phase
- **Request side-by-side screenshots** of native vs React Native TV implementations
- **Focus state comparisons** showing highlight/selection states
- **Navigation flow screenshots** demonstrating user journey preservation

## State Management Migration

**From Android ViewModels to React Native:**
```javascript
// Migration from Android ViewModel pattern
const useMovieData = () => {
  const [movies, setMovies] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchMovies();
  }, []);

  const fetchMovies = async () => {
    setLoading(true);
    try {
      const response = await movieService.getMovies();
      setMovies(response.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return { movies, loading, error, refetch: fetchMovies };
};
```

## Navigation Patterns

**From Android TV Navigation to React Native:**
- Replace Fragment navigation with React Navigation TV-optimized setup
- Implement stack navigation for details screens
- Use tab navigation for main sections
- Handle back button behavior properly for TV context

## Platform-Specific Considerations

### Android TV Specific
- Handle D-pad navigation events
- Implement proper manifest configuration
- Support for Android TV launcher requirements
- Integration with Google Assistant (voice search)

### Apple TV Specific (if targeting cross-platform)
- Handle Siri Remote gestures
- Implement tvOS-specific focus management
- Support for Apple TV App Store requirements

## Testing Strategy

**TV-Specific Testing Requirements:**
1. **Focus Navigation Testing**: Verify all elements are reachable via D-pad
2. **Performance Testing**: Monitor memory usage and frame rates
3. **Visual Regression Testing**: Compare with native app screenshots
4. **Platform Testing**: Test on both Android TV and Apple TV devices
5. **Remote Control Testing**: Verify all remote control buttons work correctly

## Code Quality Standards

- Follow React Native TV best practices
- Implement proper TypeScript typing for TV components
- Use functional components with hooks
- Implement proper error boundaries
- Follow accessibility guidelines for TV interfaces
- Maintain 60fps performance standards

## Migration Phases

### Phase 1: Foundation Setup (Request Screenshots Here)
- Set up React Native TV project with react-native-tvos
- Analyze existing app architecture and components
- Create component mapping documentation
- **Request initial app screenshots for analysis**

### Phase 2: Core Component Migration
- Implement main browse screen functionality
- Migrate content listing and grid components
- Set up basic navigation structure
- **Request screenshots during implementation for comparison**

### Phase 3: Focus Management Implementation
- Implement custom focus management system
- Add focus visual feedback and animations
- Handle edge cases and focus recovery
- **Request focus state screenshots for validation**

### Phase 4: Feature Completion
- Migrate remaining screens and functionality
- Implement platform-specific features
- Add error handling and edge cases
- **Request final comparison screenshots**

### Phase 5: Optimization and Testing
- Performance optimization for TV hardware
- Comprehensive testing across devices
- Visual parity validation with native app
- **Request final validation screenshots**

## Common Pitfalls and Solutions

1. **Focus Management Complexity**: Implement gradual focus system, test extensively
2. **Performance Issues**: Profile early, optimize images and lists
3. **Navigation Differences**: Map Android TV patterns to React Navigation
4. **Visual Inconsistencies**: Use screenshots for pixel-perfect matching
5. **Platform Differences**: Test on actual TV devices, not just simulators

## Success Criteria

- ✅ All screens and functionality ported successfully
- ✅ Focus navigation works flawlessly with D-pad/remote
- ✅ Performance matches or exceeds native app
- ✅ Visual parity with original design (verified via screenshots)
- ✅ Cross-platform compatibility (if targeting multiple TV platforms)
- ✅ Proper error handling and edge case management
- ✅ Accessibility compliance for TV interfaces

Remember: Always prioritize the 10-foot user experience, request screenshots when needed for accuracy, and ensure robust focus management as the foundation of excellent TV app experience.