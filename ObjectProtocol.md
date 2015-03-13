The underlying protocol for Microcosm is a simple object access protocol.

# Types #
  * List -- These are sent as a length (int32) followed by the contained objects.  The objects inside a list can only be of one type.
  * Tuple -- The data inside these is sent in order.  The objects can be of different types, but the length is fixed.
  * String -- This is sent as a length (int32 -- length of the data, not the number of characters) followed by UTF-8 encoded text with no terminator.
  * All your standard integer/float types are sent as-is.  Bools are sent as an 8-bit int.
  * User-defined -- Described later.

# Messages #
  1. Call:
    1. Opcode : int = 0
    1. Length : int -- Total length of the message, not counting opcode or length
    1. ObjId : int -- ID of the object containing the method to call
    1. Ordinal : int -- ID of the method to call
    1. MsgId : int -- This is the ID sent back in the return, so the appropriate callback can be called
    1. Serialized arguments
  1. Return:
    1. Opcode : int = 1
    1. Length : int -- Total length of the message, not counting opcode or length
    1. MsgId : int -- Return ID
    1. Serialized return data
  1. Free:
    1. Opcode : int = 2
    1. Length : int -- Total length of the message, not counting opcode or length
    1. ObjId : int -- Object to free
  1. DelegateCall:
    1. Opcode : int = 3
    1. Length : int -- Total length of the message, not counting opcode or length
    1. DelId : int -- ID of the delegate to call
    1. Serialized arguments

# User objects #

User-defined objects are sent as an object id (int32), which can be used as the target for a Call, can be sent as an argument/return value, etc.  Note:  As it stands, object ids sent over the wire can have their sign bit set to indicate that the object resides on the callee side.  That is, if the server sends down an object, id 4, the client can call a method and pass 0x80000004 (4 with the sign bit set) as an argument to reference the server's own object.  If the server calls a method on the client and wants to reference an object on the client side, it does the same -- the sign bit always represents an object on the callee.

When one side is done with an object from the other, a Free message containing the object id must be sent to release it.  Each side must ref-count the objects so that the Free message is only sent once per object.

# Flow #

When a client connects to a cosm, they send an initial object id (int32) and get an object id back.  All future communications take place through these objects and the objects returned by methods/properties/events therein.  To call a method, you send a Call message with the object target, ordinal (number of the method to call), and a unique message id.  Once the call is sent, a Return message will come back with the message id and the return value (if any).

# Properties #

Properties work roughly like methods, with the exception that after the msgid, you must send a 'sub-opcode' value (int8) -- 0 for get, 1 for set.  Gets take no arguments, sets take one argument (value) and have a void return.

# Events #

Events are another special case of methods.  When you call an event, you either add or remove a delegate -- a method that receives events.  After msgId, you'll send a 'sub-opcode' value (int8) -- 0 for add, 1 for remove -- followed by a delegate id (int32).  The delegate id is purely a mechanism for mapping event callbacks to the appropriate delegate, and thus must be unique on the receiving side (the side that's registering for the event).  When an event occurs, a DelegateCall message will be sent to the receiver, with the delegate id and the arguments for the event, if any.

# Packing #

Data (e.g. arguments and return values) is packed left-to-right, depth-first.  For example, `list [float * list [int] * float]` (a list of tuples containing a float, a list of ints, and another float) would be encoded as a length (the outer list), then in order, for each element of the list: a float, a length (the inner list), the integers, then another float.