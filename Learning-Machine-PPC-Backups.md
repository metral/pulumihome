On, 2018-07-16, after migrating all LM stacks which were managed by PPCs to be "hosted" stacks, we exported the DynamoDB tables for the three PPCs LM was using: stage, prod and malta.  We used the following program:

```typescript
import * as awssdk from "aws-sdk";
import * as stream from "stream";
import * as fs from "fs";

async function backupTables(tableRegion: string) : Promise<void> {
    var res: string[] = []

    const dynamodb = new awssdk.DynamoDB({ region: tableRegion });
    const tables = await dynamodb.listTables().promise();

    for (const tableName of tables.TableNames!) {
        await backupTable(tableRegion, tableName);
    }
}

async function backupTable(tableRegion: string, tableName: string): Promise<void> {
    // Create a transform stream that we'll use to convert the items in the table to an array of JSON
    // objects.
    let anyItems = false;
    const itemTransform = new stream.Transform({
        writableObjectMode: true,

        transform(chunk, encoding, callback) {
            // Perform some basic validation on the input data.
            if (chunk === null || typeof chunk !== "object") {
                callback(new Error("items must be objects"));
                return;
            }

            this.push((anyItems ? "," : "[") + JSON.stringify(chunk));
            callback();
            anyItems = true;
        },

        flush(callback) {
            this.push(anyItems ? "]" : "[]");
            callback();
        },
    });

    const fileName = `./backups/${tableName}-${Date.now()}.json`;

    console.log(`backing up ${tableName} to ${fileName}`);

    itemTransform.pipe(fs.createWriteStream(fileName));

    // Scan the table and collect all of its items into an array.
    const dynamodb = new awssdk.DynamoDB({ region: tableRegion });
    await new Promise((resolve, reject) => {
        dynamodb.scan({ TableName: tableName! }).eachPage((err: any, data: any) => {
            if (err) {
                reject(err);
                return false;
            } else if (!data) {
                itemTransform.end();
                resolve();
                return false;
            }

            for (const v of data.Items) {
                itemTransform.write(v);
            }
            process.stderr.write(".")
            return true;
        });
    });

    process.stderr.write("\n");
}

console.log(`tableRegion: ${process.argv[2]}`);

backupTables(process.argv[2]).then(s => console.log(`backups complete.`));
```

The resulting files were gziped and saved into S3 (Console URL: https://s3.console.aws.amazon.com/s3/buckets/pulumi-lm-ppc-backups/)