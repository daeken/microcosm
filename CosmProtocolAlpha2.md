# Cosm (Initial object from server) #
  1. `GetPlace(path : string) : Place`
  1. `AvatarId : int { get }` -- This will give you a unique avatar ID, which is stored in the viewer's Avatar object

# Viewer (Initial object from client) #
  1. `Avatar : Avatar { get }` -- Gets the avatar object for the viewer

# Avatar #
  1. `event PositionUpdate : CosmEventHandler [(float * float * float) * (float * float * float)]` -- The server can attach to this event to get position updates, e.g. in a multiuser env
  1. `AvatarId : int { get }` -- Unique ID for this avatar in this connection

# Place #
  1. `event PlaceUpdate : CosmEventHandler [Place]` -- Called when a Place inside the current Place is updated
  1. `event PlaceRemove : CosmEventHandler [Place]`
  1. `event RenderableUpdate : CosmEventHandler [Renderable]`
  1. `event RenderableRemove : CosmEventHandler [Renderable]`
  1. `IsMultiUser : bool { get }`
  1. `event AvatarUpdate : CosmEventHandler [int * ((float * float * float) * (float * float * float))]` -- If the Place is multi-user, this will allow viewers to get avatar information
  1. `StartLocation : (float * float * float) * (float * float * float) { get }` -- This is the starting position and orientation for avatars
  1. `Enter(viewer : Viewer) : void` -- Called on entrance
  1. `DefaultAvatarModel : Model { get }`

# Renderable #
  1. `Position : float * float * float { get }`
  1. `Orientation : float * float * float { get }`
  1. `Model : Model { get }`

# Model _(Needs texturing support)_ #
  1. `Verts : list [float * float * float] { get }`
  1. `Tris : list [(int * int * int) * (Color * Color * Color)] { get }` -- The ints are indexes into Verts.  Color is an alias for `float * float * float * float`: RGBA 0...1