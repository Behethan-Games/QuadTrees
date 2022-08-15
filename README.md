# QuadTrees

[![CircleCI](https://circleci.com/gh/splitice/QuadTrees/tree/master.svg?style=svg)](https://circleci.com/gh/splitice/QuadTrees/tree/master)

High Performance QuadTree for C# based on https://github.com/splitice/QuadTrees implementation.
This fork reworked some internals to make it compatible with structs and lowlevel operations. 

## Usage example with classes 

```CSHARP

// The objects being stored inside the quadtree
class QTreeObject: IPointFQuadStorable {

    private PointF _rect;

    public PointF Point {
	set { _rect = value; }
	get { return _rect; }
    }

    public QTreeObject(PointF rect) { _rect = rect; }
}

// The payload for lambda operations to prevent local variables copies causing allocations 
struct Payload{ public int counter; }

QuadTreePointF<QTreeObject> qtree = new QuadTreePointF<QTreeObject>();
qtree.AddRange(new List<QTreeObject>{
	new QTreeObject(new PointF(10,10)),
	new QTreeObject(new PointF(11,11)),
	new QTreeObject(new PointF(12,12)),
	new QTreeObject(new PointF(11,11)),
	new QTreeObject(new PointF(-1000,1000))
});

var amount = qtree.ObjectCount(new RectangleF(9,9,20,20));                     // Counts entities inside the range 
var list = qtree.GetObjects(new RectangleF(9, 9, 20, 20));                     // Returns new list
qtree.GetObjects(new RectangleF(9,9,20,20), list);                             // Uses existing one  

var payload = new Payload{ counter = 0; }
qtree.GetObjects(new RectangleF(9, 9, 20, 20), (obj) => { // logic });                            // Executes a lambda for each obj inside the rectangle
qtree.GetObjects(new RectangleF(9, 9, 20, 20), ref payload, (ref payload, obj) => payload++);     // Same as above, but with passed payload to pass stuff into the lambda
```

## Usage example with structs  

```CSHARP

// The objects being stored inside the quadtree
struct QTreeObject: IPointFQuadStorable {

    public int uniqueId;  // Used for hashing, just an example, any other hash method variant will work too .
    private PointF _rect;

    public PointF Point {
	set { _rect = value; }
	get { return _rect; }
    }

    public QTreeObjectStruct(int uniqueId, PointF rect) {
	this.uniqueId = uniqueId;
	_rect = rect;
    }

    public bool Equals(QTreeObjectStruct other) { return uniqueId == other.uniqueId; }

    public override bool Equals(object obj) { return obj is QTreeObjectStruct other && Equals(other); }

    public override int GetHashCode() { return uniqueId; }
}

// The payload for lambda operations to prevent local variables copies causing allocations 
struct Payload{ public int counter; }

QuadTreePointF<QTreeObject> qtree = new QuadTreePointF<QTreeObject>();
qtree.AddRange(new List<QTreeObject>{
	new QTreeObject(new PointF(10,10)),
	new QTreeObject(new PointF(11,11)),
	new QTreeObject(new PointF(12,12)),
	new QTreeObject(new PointF(11,11)),
	new QTreeObject(new PointF(-1000,1000))
});

var amount = qtree.ObjectCount(new RectangleF(9,9,20,20));                     // Counts entities inside the range 
var list = qtree.GetObjects(new RectangleF(9, 9, 20, 20));                     // Returns new list
qtree.GetObjects(new RectangleF(9,9,20,20), list);                             // Uses existing one  

var rect = new RectangleF(9, 9, 20, 20);
var count = qtree.ObjectCount(rect);
Span<QTreeObject> array = stackalloc QTreeObjec[count];           // Local array of the structs being stored in the quadtree
qtree.GetObjects(rect, array);                                    // Copy structs inside the rectangle to the local array 

var payload = new Payload{ counter = 0; }
qtree.GetObjects(new RectangleF(9, 9, 20, 20), (ref obj) => { // logic });                            // Executes a lambda for each obj inside the rectangle
qtree.GetObjects(new RectangleF(9, 9, 20, 20), ref payload, (ref payload, ref obj) => payload++);     // Same as above, but with passed payload to pass stuff into the lambda
```

## License

Since version v1.0.3 licensed under the Apache License
