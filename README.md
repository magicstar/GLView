Purpose
--------------

GLView is a collection of classes designed to make it as easy as possible to get up and running with OpenGL functionality within an iOS app.

The GLImage and GLImageView classes make it possible to load and display PVR formatted images and video clips in your app without needing to know any OpenGL whatsoever. See more about PVR images and video below.

The GLModel and GLModelView classes allow you to load a 3D model using the popular WaveFront .obj format and display it in a view, again without needing to know anything about OpenGL.

The GLView library is modular. If you don't want to render 3D models you can omit the Models classes and the rest of the library will still work. If you aren't interested in loading and displaying images and just want a basic OpenGL context set up for you, you can omit the Images and Models classes.


Supported iOS & SDK Versions
-----------------------------

* Supported build target - iOS 5.1 (Xcode 4.3.2, Apple LLVM compiler 3.1)
* Earliest supported deployment target - iOS 4.0 (Xcode 4.2)
* Earliest compatible deployment target - iOS 4.0

NOTE: 'Supported' means that the library has been tested with this version. 'Compatible' means that the library should work on this iOS version (i.e. it doesn't rely on any unavailable SDK features) but is no longer being tested for compatibility and may require tweaking or bug fixes to run correctly.


ARC Compatibility
------------------

GLView makes use of the ARC Helper library to automatically work with both ARC and non-ARC projects through conditional compilation. There is no need to exclude GLView files from the ARC validation process, or to convert GLView using the ARC conversion tool.


Installation
---------------

To use GLView, just drag the class files into your project and add the QuartzCore framework.


Classes
---------------

The GLView library currently includes the following classes:

- GLUtils - this is a collection of useful class extensions and global methods that are used by the GLView library. 

- GLView - this is a general-purpose UIView subclass for displaying OpenGL graphics on screen. It doesn't display anything by default, but if you are familiar with OpenGL you can use it for raw OpenGL drawing.

- GLImage - this is a class for loading image files as OpenGL textures. It supports all the same image formats as UIImage, as well as a number of PVR image formats.

- GLImageMap - this is a class for loading image maps, also known as image atlases or spritemaps.

- GLImageView - this is a subclass of GLView, specifically designed for displaying GLImages. It behaves in a similar way to UIImageView and mostly mirrors its interface.

- GLModel - this is a class for loading 3D mesh geometry for rendering in a GLView. It currently supports only Wavefront .obj files and the bespoke "WWDC2010" model format used in the Apple GLEssentials sample code, but will support additional formats in future.

- GLModelView - this is a subclass of GLView, specifically designed for displaying GLModels as simply and easily as possible.

- GLLight - this class represents a light source and is used for illuminating GLModel objects in the GLModelView.


Global methods
------------------

    void CGRectGetGLCoords(CGRect rect, GLfloat *coords);

This method is used by the GLView library for converting GLRects into OpenGL vertices. It receives a pointer to an array of 8 GLfloats, and it populates the array with the coordinates for the 4 corners of the rect.


NSString extensions 
--------------------

These methods extend NSString with some useful file path manipulation functionality.
    
    - (NSString *)stringByAppendingScaleSuffix;
    
This method appends an @2x suffix to the string if the device has a Retina display. The method correctly handles file extensions and device type suffixes, so the @2x will be inserted before the file extension or device type suffix if present. See the Image file suffixes section below for more information.
    
    - (NSString *)stringByDeletingScaleSuffix;
    
This method removes the @2x suffix from a file path, or does nothing if the suffix is not found.
    
    - (NSString *)scaleSuffix;
    
This method returns the @2x suffix if found, or @"" if not.
    
    - (CGFloat)scale;
    
This method returns the image scale value for a file path as a floating point number, e.g. 2.0f if the file includes an -hd suffix (on iPhone) or @2x suffix and 1.0f if it doesn't.
    
    - (NSString *)stringByAppendingInterfaceIdiomSuffix;
    
This method appends the user interface idiom suffix for the current device (~ipad or ~iphone) to the string. If the string includes a file extension, the suffix is correctly inserted before the file extension. See the Image file suffixes section below for more information.
    
    - (NSString *)stringByDeletingInterfaceIdiomSuffix;
    
This method removes the user interface idiom suffix from the string if present, or does nothing if a suffix is not found.
    
    - (NSString *)interfaceIdiomSuffix;
    
This method returns the user interface idiom suffix if found, or @"" if not.

    - (UIUserInterfaceIdiom)interfaceIdiom;

This method returns the UIUserInterfaceIdiom value specified by a file's interface idiom suffix (if found). If no suffix is found it returns the current device idiom.

    - (NSString *)stringByAppendingHDSuffix;
    
This method appends the -hd suffix to the string if the device is an iPad or Retina display iPhone and does nothing if it isn't.
    
    - (NSString *)stringByDeletingHDSuffix;
    
This method deletes the -hd suffix from the string if found, or does nothing if the suffix is not found.
    
    - (NSString *)HDSuffix;
    
This method returns @"-hd" if the string has the -hd suffix and @"" if it doesn't.
    
    - (BOOL)isHD;

This method returns YES if the string has an -hd suffix and NO if it doesn't.


    - (NSString *)absolutePathWithDefaultExtensions:(NSString *)firstExtension, ... NS_REQUIRES_NIL_TERMINATION;
    
This method takes a relative or partial file path and generates an absolute path, complete with image scale and device type suffixes (if suffixed versions of the file can be found). It uses NSFileManager to check that the file exists, so it will either return a valid file path or nil, it will never return a path to a file that does not exist. You can also specify a list of file extensions to be tried if the path does not already have an extension (these will be ignored if the file already has an extension). See the Image file suffixes section below for more information. This method makes multiple filesystem calls, so may be relatively slow the first time it is called for each path, however it caches the result for a given input so the next time it will be faster.


UIColor extensions 
---------------------

These methods extend UIColor to make it easier to integrate with OpenGL code.

    - (void)getGLComponents:(GLfloat *)rgba;
    
Pass in a pointer to a C array of four GLfloats and this method will populate them with the red, green, blue and alpha components of the UIColor. Works with monochromatic and RGB UIColors, but will fail if the color contains a pattern image, or some other unsupported color format.
    
    - (void)bindGLClearColor;

Binds the UIColor as the current OpenGL clear color.
    
    - (void)bindGLBlendColor;
    
Binds the UIColor as the current OpenGL blend color.

    - (void)bindGLColor;
    
Binds the UIColor as the current OpenGL color.


GLView properties
----------------

    @property (nonatomic, assign) CGFloat fov;

This is the vertical field of view, in radians. This defaults to zero, which gives an orthographic projection (no perspective) but set it to around Pi/2 (90 degrees) for a perspective projection.

    @property (nonatomic, assign) CGFloat near;

The near clipping plane. This is used by OpenGL to clip geometry that is too close to the camera. In orthographic projection mode, this value can be negative, but in perspective mode the value must be greater than zero.

    @property (nonatomic, assign) CGFloat far;

The far clipping plane. This is used by OpenGL to clip geometry that is too far from the camera. A smaller value for the far plane improves the precision of the z-buffer and reduces rendering artefacts, so try to set this value as low as possible (Note: it must be larger than the near value).

    @property (nonatomic, assign) NSTimeInterval frameInterval;

This property controls the frame rate of the view animation. The default value is 1.0/60.0 (60 frames per second). You may wish to reduce this to 30 fps if your animation becomes choppy.

    @property (nonatomic, assign) NSTimeInterval elapsedTime;

This property is used when animating the view. It indicates the total animation time elapsed since the startAnimating method was first called. It is not reset when stop is called, but you can reset it manually at any time as it is a read/write property.


GLView methods
----------------

GLView has the following methods:

    - (void)setUp
    
Called by `initWithCoder: and `initWithFrame:` to initialise the view. Override this method to do setup code in your own GLView subclasses. Remember to call [super setUp] or the view won't work properly.

    - (void)display;
    
If you are managing drawing yourself using an update loop, you can call this method to call the drawRect: method immediately instead of waiting for the next OS frame update.

    - (void)drawRect:(CGRect)rect;
    
This method (inherited from UIView) is responsible for drawing the view contents. The default implementation does nothing, but you can override this method in a GLView subclass to perform your own OpenGL rendering logic.

	- (void)bindFramebuffer;

This method is used to bind the GLView before attempting any drawing using OpenGL commands. This method is called automatically prior to the `drawRect:` method, so in most cases you won't ever need to call it yourself.

	- (BOOL)presentRenderbuffer;

Call this method to update the view and display the result of any previous drawing commands. This method is called automatically after the `drawRect:` method, so in most cases you won't ever need to call it yourself.

    - (void)startAnimating;
    
Begin animating the view. This will cause the `step:` method to be called repeatedly every 60th second, allowing you to do frame-by-frame animation of your OpenGL geometry.
    
    - (void)stopAnimating;
    
Pauses the animation. The elapsedTime property is not automatically reset so calling `startAnimating` again will resume the animation from the point at which it stopped.
    
    - (BOOL)isAnimating;
    
Returns YES if the view is currently animating and NO if it isn't.
    
    - (void)step:(NSTimeInterval)dt;

This method is called every 60th of a second while the view is animating. The dt parameter is the time (in seconds) since the last step (this will usually be ~1/60 seconds depending on the complexity of your rendering code and device performance). The default implementation does nothing, but you can override this method to create your own animation logic.

    - (UIImage *)snapshot;
    
This method returns the current contents of the GLView (including any subviews) as a UImage. Note that calling this method will always cause the view contents to be immediately re-rendered by calling the drawRect: method, even if the view has recently been updated. The CALayer renderInContext: method for the layer backing the GLView has also been implemented, allowing you to capture any hierarchy of views that includes GLView instances. 


GLImage properties
-------------------

	@property (nonatomic, readonly) CGSize size;
	
The size of the image, in points. As with UIImage, on a retina display device the actual pixel dimensions may be twice the size, depending on the scale property.
	
	@property (nonatomic, readonly) CGFloat scale;

The image scale. For @2x images on Retina display devices this will have a value of 2.0.

    @property (nonatomic, readonly) GLuint texture;
    
The underlying texture ID used by the GLImage. You can use this to bind the texture in your own drawing code if you do not want to use the `bindTexture` method.

    @property (nonatomic, readonly) CGSize textureSize;
    
The size of the underlying texture. The dimensions of the texture will always be a power of two. For images that have a scale of 1.0 (non-Retina) and have power-of-two dimensions, this will match the size property.

    @property (nonatomic, readonly) CGRect clipRect;

The clipping rectangle used to crop and resize the texture to fit the image rect. This rect is measured in texture pixels, so for images that are not clipped, the clipRect size will match the textureSize.

    @property (nonatomic, readonly) CGRect contentRect;

The content rectangle used to specify the portion of the image which is textured. This rect is measured in image pixels. In most cases the size of this rect will match the image size and the origin will be zero, however if the image content has been trimmed from its original size, the contentRect may be smaller than the bounds specified by the image size.

    @property (nonatomic, readonly) BOOL premultipliedAlpha;

Images that have translucent parts can either use premultiplied or non-premultiplied alpha. iOS typically uses premultiplied alpha when loading images and this is the default for non-PVR images. PVR images generated using Apple's command-line tools do not have premultiplied alpha, so for PVR images it is assumed that the image does not have premultiplied alpha so this property will be NO for PVR images by default, however some tools have the option to generate PVR images with premultiplied alpha, and this is generally recommended to avoid odd black or white halos around opaque parts of the image. There is no way to detect if a PVR image was generated with premultiplied alpha, so if you know that it was, or if the image looks wrong when rendered, you can toggle this property using the `imageWithPremultipliedAlpha:` method. See the Premultiplied Alpha section below for more details.

    @property (nonatomic, readonly) const GLfloat *textureCoords;

The texture coordinates used for rendering the image. These are handy if you need to render the image yourself using OpenGL functions instead of using the `drawAtPoint:` or `drawInRect:` methods. The textureCoords array will always contain exactly 8 GLfloat values.

    @property (nonatomic, readonly) const GLfloat *vertexCoords;

The vertex coordinates used for rendering the image. These are handy if you need to render the image yourself using OpenGL functions instead of using the `drawAtPoint:` or `drawInRect:` methods. The vertexCoords array will always contain exactly 8 GLfloat values.


GLImage methods
------------------

	+ (GLImage *)imageNamed:(NSString *)nameOrPath;
	
This method works more-or-less like the equivalent UIImage method. The image file specified by the nameOrPath paramater will be loaded and returned. The image is also cached so that any subsequent `imageNamed` calls for the same file will return the exact same copy of the image. IN a low memory situation, this cache will be cleared. Retina display images using the @2x naming scheme also behave the same way as for UIImage, as do images that have the ~ipad or ~iphone suffix.

The name can include a file extension. If it doesn't, .png is assumed. The name may also include a full or partial file path, and, unlike UIImage's version, GLIMage's imageNamed function can load images outside of the application bundle, such as from the application documents folder. Note however that because these images are cached, it is unwise to load images in this way if they are likely to be replaced or deleted while the app is running, as this may result in unexpected behaviour.

**NOTE:** OpenGL textures are required to have dimensions that are a power of two, e.g. 8x128, 32x64, 128x128, 512x256, etc. As of version 1.2.2, GLImage can load images whose dimensions are not powers of two and it will automatically frame them inside the smallest valid texture size. However, it is still recommended that you try to use images that are powers of two whenever possible, as it avoids wasting video memory.
	
	+ (GLImage *)imageWithContentsOfFile:(NSString *)nameOrPath;
	- (GLImage *)initWithContentsOfFile:(NSString *)nameOrPath;

These methods load a GLImage from a file. The path parameter can be a full or partial path. For partial paths it is assumed that the path is relative to the application resource folder. If the file extension is omitted, it is assumed to be .png. Retina display images using the @2x naming scheme behave the same way as for UIImage, as do images that have the ~ipad or ~iphone suffix. Images loaded in this way are not cached or de-duplicated in any way.
	
	+ (GLImage *)imageWithUIImage:(UIImage *)image;
	- (GLImage *)initWithUIImage:(UIImage *)image;
	
These methods create a GLImage from an existing UIImage. The original scale and orientation of the UIImage are preserved. The result GLIImage format will be a 32-bit uncompressed RGBA texture.
	
    + (GLImage *)imageWithSize:(CGSize)size scale:(CGFloat)scale drawingBlock:(GLImageDrawingBlock)drawingBlock;
    - (GLImage *)initWithSize:(CGSize)size scale:(CGFloat)scale drawingBlock:(GLImageDrawingBlock)drawingBlock;

These methods allow you to create a new GLImage by drawing the contents directly using Core Graphics. The first and second arguments specify the size and scale of the image, and the third argument is a block function that you use to do your drawing. The block takes a single argument which is the CGContextRef for you to draw into. See the "Drawing" tab in the GLImage Demo example app for a demonstration.

    + (GLImage *)imageWithData:(NSData *)data scale:(CGFloat)scale;
    - (GLImage *)initWithData:(NSData *)data scale:(CGFloat)scale;

These methods allow you to create a GLImage from an NSData object. The data content should correspond to an image file in one of the formats supported by `imageWithContentsOfFile:`. The image size and format is derived automatically from the data. The scale parameter can be used to indicate the display resolution that the image is optimised for.

    - (GLImage *)imageWithPremultipliedAlpha:(BOOL)premultipliedAlpha;
    
Images that have translucent parts can either use premultiplied or non-premultiplied alpha. iOS typically uses premultiplied alpha when loading images and this is the default for non-PVR images. PVR images generated using Apple's command-line tools do not have premultiplied alpha, so for PVR images it is assumed that the image does not have premultiplied alpha. Some tools however have the option to generate PVR images with premultiplied alpha, and this is generally recommended to avoid odd black or white halos around opaque parts of the image, but since there is no way to detect this from the file format, these images may render incorrectly when loaded with GLImage. To correct this, use this method with a value of YES to create a version of the image that will render correctly. See the Premultiplied Alpha section below for more details.

    - (GLImage *)imageWithOrientation:(UIImageOrientation)orientation;

If your image is flipped or was rotated to fit into a larger image map, you can change the orientation at which it is displayed by updating the orientation with this method. This method doesn't change the image pixels, it only affects the way in which it is displayed by the `drawAtPoint:` and `drawInRect:` methods.
    
    - (GLImage *)imageWithClipRect:(CGRect)clipRect;
    
This method can be used to set the clipping rect for the image. This is useful if you are loading a PVR texture image where the content is smaller than the image bounds but, due to the PVR format restrictions, the image file has to be a square with power-of-two sides. Note that the clipping rect is measured in texture pixels, not in image coordinates, and does not take into account the scale factor or previous clipping. You can determine the actual texture Size from the textureSize property of the image. **Note:** Adjusting the clipRect will also adjust the image size and contentRect accordingly, however you can override this by calling `imageWithSize:` and/or `imageWithContentRect:` afterwards.

    - (GLImage *)imageWithContentRect:(CGRect)contentRect;
    
This method can be used to set the content rect for the image. This is useful if you want to manipulate the image's anchor point, or add padding around the textured part of the image.

    - (GLImage *)imageWithScale:(CGFloat)scale;
    
This method can be used to set the image scale. Modifying the scale will automatically update the image size and contentRect accordingly, but will not actually change the image pixels. In the unlikely case that you wish to set the scale without modifying the image size, you can override call `imageWithSize:` afterwards to restore the original size.

    - (GLImage *)imageWithSize:(CGSize)size;
    
This method can be used to set the image size. This method does not actually modify the image pixels, it merely changes the default horizontal and vertical scale factor at which the image is drawn when using the `drawAtPoint:` method.

	- (void)bindTexture;
	
This method is used to bind the underlying OpenGL texture of the GLImage prior to using it for OpenGL drawing.
	
	- (void)drawAtPoint:(CGPoint)point;
	
This method will draw the image into the currently bound GLView or OpenGL context at the specified point.
	
	- (void)drawInRect:(CGRect)rect;

This method will draw the image into the currently bound GLView or OpenGL context, stretching it to fit the specified CGRect.


GLImageMap methods
----------------------

The GLImageMap class has the following methods:

    + (GLImageMap *)imageMapWithContentsOfFile:(NSString *)nameOrPath;
    - (GLImageMap *)initWithContentsOfFile:(NSString *)nameOrPath;
    
These methods are used to create a GLImageMap from a file. The parameter can be an absolute or relative file path (relative paths are assumed to be inside the application bundle). If the file extension is omitted it is assumed to be .plist. Currently the only image map file format that is supported is the Cocos2D sprite map format. GLImageMap fully supports trimmed, rotated and aliased images. As with ordinary GLImages, GLImageMap will automatically detect @2x retina files and files with the ~ipad or ~iphone suffix.

    + (GLImageMap *)imageMapWithImage:(GLImage *)image data:(NSData *)data;    
    - (GLImageMap *)initWithImage:(GLImage *)image data:(NSData *)data;
    
These methods are used to create a GLImage from data. The data should represent the contents of an image map file in one of the formats supported by the `imageMapWithContentsOfFile:` method. If the image argument is nil, GLImageMap will attempt to locate the texture file from the filename specified in the data, however if the image file is not located in the root of the application bundle, it may not be able to find it. In this case, you can supply a GLImage to be used as the image map image and the image file specified in the data will be ignored.
    
    - (NSInteger)imageCount;
    
This method returns the number of images in the image map.
    
    - (NSString *)imageNameAtIndex:(NSInteger)index;
    
This method returns the image name at the specified index. Note that image map images are unordered, so do not assume that the image order will match the order of images in the file that was loaded. If you wish to access image map images in a specific order, it is a good idea to name them numerically.
    
    - (GLImage *)imageAtIndex:(NSInteger)index;
    
This method returns the image map image at the specified index. Note that image map images are unordered, so do not assume that the image order will match the order of images in the file that was loaded. If you wish to access image map images in a specific order, it is a good idea to name them numerically.
    
    - (GLImage *)imageNamed:(NSString *)name;
    
This method returns the image map image with the specified name. Depending on the tool used to generate the image map data file, the names may include a file extension. If you do not include a file extension in the image name, png is assumed.


GLImageView properties
-----------------------

The GLImage view has the following properties:

	@property (nonatomic, strong) GLImage *image;

This is used to set and display a GLImage within the view. The standard UIView `contentMode` property is respected when drawing the image, in terms of how it is cropped, stretched and positioned within the view.

    @property (nonatomic, strong) UIColor *blendColor;
    
An (optional) color to blend with the image before it is rendered. This can be used to tint the image, or modify its opacity, etc. Defaults to white.

	@property (nonatomic, copy) NSArray *animationImages;
	
This is used to specify a sequence of images to be played as an animation. The array can contain either GLImages, or filenames. If filenames are supplied then the images will be streamed from disk rather than being loaded in advance. See the Video tab of the GLImage Demo example app for a demonstration of how this can be used to play a large PVR video sequence with smooth 30fps performance and minimal memory consumption. 
	
	@property (nonatomic, assign) NSTimeInterval animationDuration;

The duration over which the animationImages sequence will play. This is measured in seconds and defaults to the number of animation frames divided by 30 (i.e. 30 frames-per-second). This value is automatically set whenever the animationImages array is updated, so remember to set this *after* you set the animationImages property.

	@property (nonatomic, assign) NSInteger animationRepeatCount;

The number of times the animation will repeat. Defaults to zero, which causes the animation to repeat indefinitely until the `stopAnimating` method is called.


GLImageView methods
-----------------------

	- (GLImageView *)initWithImage:(GLImage *)image;
	
This method creates a GLImageView containing the specified image and sets the view frame to match the image size.

	- (void)startAnimating;
	
Inherited from GLView, this method starts the `animationImages` sequence. Playback always starts from the first frame. Calling play when the animation is already playing will start it again from the beginning.
	
    - (void)stopAnimating;

This method stops the animation sequence. Once stopped, the sequence can not be resumed, only started again from the beginning.

    - (BOOL)isAnimating;

This method returns YES if the `animationImages` sequence is currently playing.


GLModel methods
--------------------

    + (GLModel *)modelWithContentsOfFile:(NSString *)path;
    - (GLModel *)initWithContentsOfFile:(NSString *)path;
    
These methods load a GLModel from a file. The path parameter can be a full or partial path. For partial paths it is assumed that the path is relative to the application resource folder. The format is inferred from the file extension; Currently only .obj (Wavefront) and .model (Apple's GLEssentials sample code model format) files are accepted. Models loaded in this way are not cached or de-duplicated in any way. Note that is is also possible to use the @2x and ~ipad/~iphone filename suffixes to specify different models for different devices, which is useful if you wish to provide more detailed models for higher-end devices.

    - (GLModel *)initWithContentsOfFile:(NSString *)path;
    - (GLModel *)initWithData:(NSData *)data;

These methods initialise a model with data. The format of the data should match the contents of one of the supported file formats.

    - (void)draw;

Renders the model in the current GLView.  In practice you may wish to configure the OpenGL state for the model before calling draw, e.g. by setting a texture image to use for the rendering. See the GLModelView `layoutSubviews` method for an example.


GLModelView properties
-----------------------

    @property (nonatomic, strong) GLModel *model;
    
The model that will be rendered by the view.
    
    @property (nonatomic, strong) GLImage *texture;
    
A GLImage that will be used to texture the model. The model must have texture coordinates defined in order for the texture to be applied correctly.
    
    @property (nonatomic, strong) UIColor *blendColor;
    
A color to blend with the model texture. This can be used to tint the model or modify its opacity. If no texture image is specified, the model will be flat-shaded in this color.

    @property (nonatomic, copy) NSArray *lights;
    
An array of GLLight objects used to illuminate the model. By default this array contains a single white light positioned above and to the left of the object. You can set up to eight lights, although increasing the number of lights has a negative impact on performance. Setting an empty array is equivalent to having a single ambient white light.
    
    @property (nonatomic, assign) CATransform3D transform;
    
A 3D transform to apply to the model. This can be used to center, scale or rotate the model to fit the view frame. See the example project for how this can be used.


GLLight properties
---------------------

    @property (nonatomic, strong) UIColor *ambientColor;
    
The ambient light color. This color is used to illuminate the object uniformly. Defaults to black (off).
    
    @property (nonatomic, strong) UIColor *diffuseColor; 
    @property (nonatomic, strong) UIColor *specularColor;
    
The diffuse and specular light colors. These illuminate the object in a directional fashion, eminating from the position specified by the transform. These default to white (full brightness).
    
    @property (nonatomic, assign) CATransform3D transform;

The transform property controls the position of the light. Currently only the position/translation part of the transform is used. Rotation and scale properties are ignored. Use CATransform3DMakeTranslation(x, y, z) to set a suitable value. The default translation is (0, 0, 1), which is typically behind the camera, or between the camera and the scene being viewed.


GLLight methods
---------------------

    - (void)bind:(GLuint)light;

This method binds the GLLight to a particular light in the scene. The light parameter is an OpenGL constant representing the light index, where GL_LIGHT0 is the first available light, and GL_LIGHT7 is typically the last available light.


Image file suffixes
--------------------

iOS has a clever mechanism for managing multiple versions of assets by using file suffixes. The first iPad introduced the ~ipad suffix for specifying ipad-specific versions of files (e.g. foo~ipad.png). The iPhone 4 introduced the @2x suffix for managing double-resolution images for Retina displays (e.g. foo@2x.png). With the 3rd generation iPad you can combine these to have Retina-quality iPad images (e.g. foo@2x~ipad.png).

This is an elegant solution for apps, but is sometimes insufficient for games because, unlike apps, hybrid games often share near-identical interfaces on iPhone and iPad, with the assets and interface elements simply scaled up, and this means that the standard definition iPad and Retina resolution iPhone need to use the same images.

Naming your images with the @2x suffix works for iPhone but not iPad, and naming them with the ~ipad suffix works for iPad but not iPhone, which forces you to either duplicate identical assets with different filenames, or to write your own file loading logic.

The -hd suffix is a concept introduced by the Cocos2D library to the problem of wanting to use the same 2x graphics for both the iPhone Retina display and the iPad standard definition display by using the same -hd filename suffix for both.

GLView supports this solution by adding some utility methods to NSString for automatically applying this suffix to file paths. All of GLView's image loading routines inherit this logic so that files using the @2x, ~ipad and -hd conventions (or any combination thereof) are automatically detected and loaded as appropriate. For example, If you load attempt to load an image called foo.png, GLImage will automatically look for foo@2x.png or foo-hd.png on a Retina iPhone, or foo~ipad.png or foo-hd.png on an iPad and will also find foo-hd@2x.png if you are using a Retina iPad.


PVR images
--------------

The PVR image format is a special proprietary image format used by the PowerVR graphics chip in iOS devices. PVR images load quicker and can take up less space in memory than PNGs, so they can be very useful for apps that need to load and display a lot of images.

Due to the rapid loading, they can also be streamed off disk fast enough to create movie clips, which can be a handy way to display video if the standard movie player APIs don't meet your requirements (e.g. if you need video with transparent portions).

You will probably notice when creating PVRs that the file size is actually larger than the equivalent PNG file. This is misleading however - PNG files use internal zip compression which makes them small on disk, but when they are loaded into memory they are expanded to consume an amount of space equivalent to their width * height * 4 bytes.

The same is true of JPEG or any other image format. But with PVR, the size on disk matches the size in memory, so a 16-bit PVR only consumes half as much memory as a PNG of the same dimensions, and a compressed PVR takes up even less space. Since the structure on disk is exactly the same as in memory, PVRs also load quicker because there is no need to decompress or transcode them.

PVRs are restricted to square images with power-of-two sizes however. See the note below about conversion.
	

Generating PVR image files
----------------------------

GLImage can load PVR images with any of the following pixel formats:

- RGBA8888 - 32 bit, high quality 24-bit colour with 8-bit alpha
- RGBA4444 - 16 bit, 12-bit colour and 4-bit alpha
- RGBA5551 - 16 bit, higher precision 5-bit color but only 1-bit alpha
- RGB565 - 16 bit, higher precision color, but no alpha transparency
- PVRTC4 - 4 bits-per-pixel lossy compression, with or without alpha
- PVRTC2 - 2 bits-per-pixel lossy compression, with or without alpha

Apple includes a command-line PVR texture generation application called texturetool with the Xcode developer tools. This can usually be found in the following easy-to-remember location:

    /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/texturetool

You may wish to set up an alias to this tool by typing the following into the terminal:

    alias texturetool='/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/texturetool'

The texturetool application is fairly limited, and can only be used to create 4bpp (bits-per-pixel) and 2bpp compressed images, which are extremely low quality and look a bit like highly compressed jpegs. These are probably not good enough for detailed still images, especially images containing transparency (the PVR OpenGL logo in the example demonstrates this), but may be appropriate for video frames or textures for 3D models.

**NOTE:** In addition to needing power-of-two dimensions, PVR images must also be perfectly square, i.e. the width and height must be equal. Valid sizes are 2x2, 4x4, 8x8, 16x16, 32x32, 64x64, 128x128, 256x256, etc. Remember to crop or scale your images to a valid size before converting them.

The typical texturetool settings you will want to use are one of the following:

	texturetool -e PVRTC --channel-weighting-perceptual --bits-per-pixel-4 -f PVR -o {output_file_name}.pvr {input_file_name}.png

This generates a 4 bpp compressed PVR image.

	texturetool -e PVRTC --channel-weighting-perceptual --bits-per-pixel-2 -f PVR -o {output_file_name}.pvr {input_file_name}.png
	
This generates a 2 bpp compressed PVR image.

As stated previously, these files will appear like heavily compressed JPEG images, and will probably not be appropriate for user interface components or images with fine detail. It is also possible to create PVR images in a variety of higher qualities, but for this you will need to use a different tool. One such app is TexturePacker (http://www.texturepacker.com/), which is not free, but provides a handy command line tool for generating additional PVR formats.

The typical TexturePacker settings you will want to use are one of the following:

	TexturePacker --disable-rotation --no-trim --opt RGBA8888 {input_file_name}.png --sheet {output_file_name}.pvr
	
This creates a 32-bit maximum-quality PVR image with alpha transparency.

	TexturePacker --disable-rotation --no-trim --dither-fs-alpha --opt RGBA4444 {input_file_name}.png --sheet {output_file_name}.pvr
	
This creates a 16-bit dithered PVR image with alpha transparency.

	TexturePacker --disable-rotation --no-trim --dither-fs --opt RGB565 {input_file_name}.png --sheet {output_file_name}.pvr
	
This creates an opaque 16-bit dithered PVR image without alpha transparency.


Generating PVR video clips
----------------------------

To use the GLImageView as a PVR video player, you'll need to convert your video into a sequence of PVR images. Assuming you are starting with a Quicktime-compatible video file, follow these steps. If you've already got your video as a sequence of numbered PNG or JPEG images then you can skip to step 4.

1) Open your video in QuickTime 7 (not Quicktime X). You'll need a QuickTime Pro license to do anything useful.

2) Assuming the video isn't already a square with power-of-two sides, you first need to convert it to the correct aspect ratio. Select Export... and choose 'Movie to MPEG-4'. Within this interface you can select a custom width and height. Go for the nearest square power-of-two dimensions to the actual video size, and you're probably better off choosing to crop or scale the image than letterbox it. Go for the highest quality you can to avoid compression artefacts.

3) After exporting your video as an MP4, open the new video in QuickTime 7 and go to the export option again. This time select 'Movie to Image Sequence' and export the video as a sequence of PNG files.

4) Now that you have the individual frames, you'll need to convert them to PVRs. Using texturetool or TexturePacker you can batch convert the images. Here are some example command line options:

	find {image_directory} -name \*.png | sed 's/\.png//g' | xargs -I % -n 1 texturetool -e PVRTC --channel-weighting-perceptual --bits-per-pixel-4 -f PVR -o %.pvr %.png
	
This generates the frames as 4 bpp compressed PVR images (This will take a while). You can do the same with TexturePacker as follows:

	find {image_directory} -name \*.png | sed 's/\.png//g' | \
    xargs -I % -n 1 TexturePacker %.png --disable-rotation --no-trim --dither-fs-alpha --opt RGBA4444 %.png --sheet %.pvr

This would create the frames as maximum-quality 32-bit PVR images with alpha transparency.

**NOTE:** PVR image sequences are very large compared with the original MP4 movie, or even the equivalent PNG image sequence. PVR is optimised for memory usage and loading speed, not disk space, so be prepared for your app to grow dramatically if you include a lot of PVR video frames.


Premultiplied Alpha
---------------------

Images with an alpha channel can be generated with either straight or premultiplied alpha. Typically on iOS, images use premultiplied alpha because it is more efficient for rendering. PNG images added to your Xcode project will be automatically converted to use premultiplied alpha, so GLImage assumes premultiplied alpha when loading ONG images.

Oddly, the texturetool that Apples ships with Xcode for creating compressed PVR images generates straight alpha, so GLImage assumes straight alpha for PVR images.

Straight alpha can result in a one-pixel black or white halo around the opaque parts of the image when rendered (you can see slight white halos in the PVR images in the GLImage demo), so it's recommended that you use premultiplied alpha with PVR images, however to generate PVR images with premultiplied alpha you'll need to use another tool such as TexturePacker or Zwoptex to generate your PVRs.

There's no way to automatically detect the type of alpha when loading a PVR image, so even if you have generated your PVRs with premultiplied alpha, GLImage will still assume they are straight multiplied. To correct this call `imageWithPremultipliedAlpha:` on the image after loading it to create a copy that will treat the image as having premultiplied alpha instead (or vice versa).

The Cocos2D Plist file format supported by GLImageMap includes metadata indicating whether the texture file has premultiplied alpha, so no further work is needed when loading sprite sheets with GLImageMap.